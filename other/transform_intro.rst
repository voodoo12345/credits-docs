Credits Transform is the core component used to define functional logic in a Credits Blockchain. A populated Transform
is then combined with one or more `Proofs <%7B%%20post_url%202016-07-31-glossary%%7D#proofs>`__, in order to make a full
Transaction which a blockchain network can process.

When implementing a Transform, you minimally implement five methods: marshall, unmarshall, required\authorizations,
verify, and apply.

Together, these five methods define the required data, validation logic, and application logic in a way the Credits
Framework can understand and utilize.

The example below will show how to implement a simple token value transfer. You can follow along below or work from the
complete example found `here <./examples/simple_value_transform.py>`__.

Getting Started
---------------

Our first step when implementing a Transform is to inherit from the ``Transform`` base class and assign an FQDN for this
transform, in this example the FQDN will be ``works.credits.value.Transfer``. This FQDN should be unique within a given
blockchain network and will ideally be globally unique.

.. code:: python

    from credits.transform import Transform

    class ValueTransform(Transform):
        FQDN = "works.credits.value.Transfer"

A transform operates with user-provided data. Our constructor should populate the object with the user-provided payload.

.. code:: python

        def __init__(self, from_address, to_address, amount):
            self.from_address = from_address
            self.to_address = to_address
            self.amount = amount

As this transfer is nice and simple and simply debits and amount from the sender and credits it to the receiver, we only
need a few fields populated.

    **Note**

    The data fields should only accept integers, strings, lists, or dictionaries. This will be implicitly enforced by
    the framework and any input that does not conform to this will be discarded while being onboarded to the DApp.

Marshalling
-----------

Our Transform object needs methods to enable reading and writing from stream. This allows it to be written to file or
network easily.

.. code:: python

        def marshall(self):
            return {
                "fqdn": self.FQDN,
                "from_address": self.from_address,
                "to_address": self.to_address,
                "amount": self.amount,
            }

        @classmethod
        def unmarshall(cls, registry, payload):
            return cls(
                from_address=payload["from_address"],
                to_address=payload["to_address"],
                amount=payload["amount"],
            )

It is critical to ensure you include all relevant fields in these two methods as well as including the FQDN in the
marshalled output.

    **Note**

    The Transform base class also includes a ``hash`` method which, by default, takes the output of the marshalled
    object and generates a hash based on the contents.

    A ``get_challenge`` method is also included and is used by Proofs as their authentication challenge. By default,
    this method directly passes through to ``hash``.

Required Authorizations
-----------------------

The next step is to declare what authorizations this transform requires to be valid. Any addresses in this list must
provide a matching proof for the transform to be cryptographically valid.

In this basic example, the only party required to authorize the Transform is the party sending value from their account.

.. code:: python

        def required_authorizations(self):
            return [self.from_address]

    **Note**

    A Transaction wrapper object will consist of one Transform and one or more Proofs. For each address returned by the
    ``required_authorizations`` method, a corresponding Proof must be present for a Transaction to be valid.

Verification
------------

The final step is to do the business logic, beginning with the ``verify`` method.

This method is called to validate that a Transform is qualified to be applied to a given state object. In this simple
example, the check simply ensures they sender has a sufficient balance to send the amount they are attempting to send.

.. code:: python

        def verify(self, state):
            # Get the state we care about
            balances = state["works.credits.balances"]

                # Check if the sender exists.
            if self.from_address not in balances.keys():
                error = "%s does not exist in %s" % (self.from_address, self.__STATE_BALANCES)
                self.logger.error(error)
                return None, error

                # Check if the sender has sufficient balance to make the transfer.
            if balances[self.from_address] < self.amount:
                error = "%s does not have enough funds (%d) to perform this transfer (%d)." % (
                    self.from_address,
                    balances[self.from_address],
                    self.amount,
                )
                self.logger.error(error)
                return None, error

            return state, None

Keep in mind this method does not check for any necessary authorizations as this is performed by the Proof ``verify``
method and is generally called by the Transaction wrapper object immediately after the Transform verification returns
successfully.

    **Note**

    -  While we do return the provided state option for convenience, the ``verify`` method should not actually update
       state.

    -  The logger object is stored as an attribute to the base Transform class and is available for logging any events
       in the manner shown above.

Application
-----------

Our final step in implementation is to define the ``apply`` method.

This method takes the current state of the network as an argument and returns an updated state with the changes
specified in the Transform's payload applied.

.. code:: python

        def apply(self, state):
            # Get the state we care about
            balances = state["works.credits.balances"]
            # Debt the sender
            balances[self.from_address] -= self.amount
            # Credit the receiver
            balances[self.to_address] += self.amount

            # return the final state
            return state, None

    **Note**

    This method is called by the Transaction wrapper object and is always immediately followed by a call to the
    ``apply`` method of all attached Proofs.

    All calls triggered by the Transaction object are applied atomically, so if any of the calls fail, the state object
    rolls back all changes made by any of the objects.

We now have a completed Transform that allows users to transfer tokens in any blockchain where the Transform is enabled.

Additional Considerations
-------------------------

**importing libraries**

If you wish to use another library within a Transform, import the library within the class itself and assign it to an
internal attribute, rather than importing the library into the global namespace.

**Post-Block Hooks**

In the near future, we will be exposing the capability to implement time and event driven functionality.

This will be done using a mechanism we have termed *Post-Block Hooks*, which allows you to build Transforms that add a
job to a queue when applying a Transform and defining events that result in execution of that job.

This allows functionality such as derivatives contracts, prediction markets, and recurring transactions.

When released, this capability will be announced on our mailing list and on our Knowledge Base.
