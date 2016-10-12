.. _transactions-transforms-proofs:

Transactions, transforms, proofs
================================

.. _transaction:

Transaction
^^^^^^^^^^^

The Transaction is an :ref:`Applicable <interfaces-applicable>`,
:ref:`Marshallable <interfaces-marshallable>`, and :ref:`Hashable <interfaces-hashable>`
object. Transactions are the units of work used to manipulate state on the blockchain.
Transactions consist of always one ``Transform`` to mutate state and one-or-many ``Proofs`` to authenticate
the changes proposed in the ``Transform``.

To construct a ``Transaction`` its dependencies should be constructed and injected.
First, a ``Transform`` must be constructed, each transform's constructor
will differ based on its behaviour. The transforms are what will essentially
define the business logic of the application.

Once the ``Transform`` has been created, at least one Proof needs to be added to
the transaction. Use ``Transform.hash()`` method to generate a challenge for the Proof,
and ``Proof.sign(key)`` method to sign the transaction.
Note that a ``Transform`` has a ``required_authorizations`` method which
will return a list of mandatory addresses that must provide a Proof.

.. code-block:: python
   :linenos:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from credits.transform import BalanceTransform
    from credits.proof import SingleKeyProof
    from credits.key import ED25519SigningKey
    from credits.address import CreditsAddress
    from credits.hash import SHA256HashProvider
    
    HASH_PROVIDER = SHA256HashProvider()
    
    ALICE_SK = ED25519SigningKey.new()
    ALICE_VK = ALICE_SK.get_verifying_key()
    ALICE_ADDR = CreditsAddress(ALICE_VK.to_string()).get_address()
    
    BOB_SK = ED25519SigningKey.new()
    BOB_VK = BOB_SK.get_verifying_key()
    BOB_ADDR = CreditsAddress(BOB_VK.to_string()).get_address()
    
    transform = BalanceTransform(ALICE_ADDR, BOB_ADDR, 100)
    # transform.required_authorizations() ==  [ALICE_ADDR]
    
    transaction = Transaction(
        transform=transform,
        proofs=[
            SingleKeyProof(ALICE_ADDR, 0, transform.hash(HASH_PROVIDER)).sign(ALICE),
        ]
    )
    
    payload = transaction.marshall()  # This output can be jsonified and POSTed to /api/v1/transaction


.. _transform:

Transform
^^^^^^^^^

Transform is an :ref:`Applicable <interfaces-applicable>`,
:ref:`Marshallable <interfaces-marshallable>` and :ref:`Hashable <interfaces-hashable>`
object. Transforms are constructed with all necessary values required to modify
state during the ``apply()`` method call and are used as a standard unit of work
inside the Credits Framework to modify :ref:`Blockchain State <blockchain-state>`.

When a Transaction with Transform inside is onboarded to the Node, it is verified against the current
:ref:`Blockchain State <blockchain-state>`. The ``verify()`` method should perform
checks against the state to check if this Transform will be applicable either
now or sometime in the future. If a Transform fails verification at any point it
will be flushed from the Node. Note that if a Transform attempts to modify state
during its validation, changes made to the state will be disposed of.

When a Transform is applied it will be given the current state of the Network
and expected to modify and return a new state. During the
:ref:`transaction application<blockchain-applying-transactions>`, a Transform may perform
any verification that has to be performed "upon application". If this verification fails,
the apply should fail and return an erroneous result. However, failure of ``apply`` doesn't
cause the transaction to be discarded. It stays in the unconfirmed pool until it's
either gets confirmed or it's ``verify`` method also fails. Only when ``verify`` fails
transaction is discarded and forgotten.


Hash storage transform
----------------------

Hash storage use case is probably the simplest one possible on the blockchain.
In this case following transform can be used:

.. code-block:: python
   :linenos:

    class LogHashTransform(Transform):
        STATE_BALANCE = "core.credits.log.hashes"

        def __init__(self, hash):
            self.hash = hash

        def verify(self, state):
            if state[self.LOG_STATE][self.hash]:
                return None, "Already have this hash logged!"
            return None, None

        def apply(self, state):
            state[self.LOG_STATE][self.hash] = {"logged_at": time.asctime()}
            return state, None


This transform will first verify the hash is not already loaded. If it is loaded
then it fails. When it comes to the application then it simply sets the hash against
the time it was applied to the state of the world. Taking this idea a more
complex KYC or logging system could easily be developed.


Balance transfer transform
--------------------------

Here is an example implementation of a simple balance transfer transform. It
implements ``credits.transform.Transform`` interface and required sanity checks
for transferring basic token balances between accounts.

.. code-block:: python
   :linenos:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from credits import transform
    from credits import stringify
    from credits import test

    """
    In this example we create a basic "Balance Transfer" transform. We then use
    the credits.test.check_transform() to validate that all expected
    attributes/methods/behaviours are provided.
    """

    class BalanceTransform(transform.Transform):
        fqdn = "credits.test.BalanceTransform"

        def __init__(self, addr_from, addr_to, amount):
            self.addr_from = addr_from
            self.addr_to = addr_to
            self.amount = amount

        def marshall(self):
            return {
                "fqdn": self.fqdn,
                "addr_to": self.addr_to,
                "addr_from": self.addr_from,
                "amount": self.amount,
            }

        @classmethod
        def unmarshall(cls, registry, payload):
            return cls(
                addr_from=payload["addr_from"],
                addr_to=payload["addr_to"],
                amount=payload["amount"],
            )

        def verify(self, state):
            """
            Verify it is possible, to apply either now or in future. Return an
            errornous response if verification fals.
            """
            balances = state["credits.test.Balances"]

            if self.addr_from not in balances:
                return None, "%s not in credits.test.Balances."

            if self.amount <= 0:
                return None, "amount must be greater than 0."

            if balances[self.addr_from] < self.amount:
                return None, "%s does not have balance to make transfer."

            return None, None  # valid transaction

        def apply(self, state):
            try:
                balances = state["credits.test.Balances"]
                balances[self.addr_from] -= self.amount
                balances[self.addr_to] = balances.get(self.addr_to, 0) + self.amount  # addr_to might not exist.
                return state, None  # return the new state.

            except Exception as e:
                return None, e.args[0]  # Something went really wrong, don't apply.

        def hash(self, hash_provider):
            return hash_provider.hexdigest(stringify.stringify(self.marshall()))

You can find this example in balance_transform.py_.

.. _balance_transform.py: https://github.com/CryptoCredits/credits-common/blob/develop/examples/balance_transform.py

In this example ``verify`` method references to *now or in future*, this is because of the way
the ``verify`` and ``apply`` logic is working in conjunction with the unconfirmed transactions
pool. The ``verify`` is called against current global state once the transaction
is trying to be onboarded, and if it passes (i.e. doesn't return an error) - the transaction
is onboarded into node's unconfirmed transaction pool. However at this point, the ``apply``
is not yet invoked. Once the node will attempt to put the transaction into a block it will call
the transform's ``apply`` method, and that method may have it's own additional verification logic.
For example, the simplest case can be the proof's nonce check. In the transaction's ``verify``
method the nonce expected to be equal or greater than the current nonce recorded in the global state.
That mean the nonce can be just next one in line, or far greater than the one in global state.

However, the ``apply`` logic by default requires transaction nonce to be exactly equal to global state,
so the transaction with nonce far off will fail to apply. In this situation, the
transaction that will successfully ``verify`` but will fail to ``apply`` will hang
in the unconfirmed transactions pool until the time for it will come, or until
``verify`` itself will fail and the transaction will be discarded.

Understanding this nuanced mechanics allows creating customised behaviours with
complex future dependencies and delayed execution.

.. _proof:

Proof
^^^^^

The Proof is an :ref:`Applicable <interfaces-applicable>` object requiring both ``verify`` and
``apply`` methods implemented. Proofs are constructed with some sort of resolvable
address, a nonce (which is typically an autoincrementing number), and a
challenge to sign. This challenge will typically be the hash of a Transform.

Once constructed a Proof is *unsigned* and a ``sign`` method must be called
with a ``signing_key`` to generate a ``verifying_key`` and ``signature``. Once
signed a Proof is now considered valid as during it's ``verify`` call it will
attempt to convert the ``verifying_key`` into an address. This address will be
compared to the address the Proof was constructed with.

When Proofs are onboarded to the Node as a part of Transaction, they are verified
against State to check that a Signature exists as well as any proof specific
ordering is valid. If a Proof is onboarded in an unsigned state it's parent
Transaction will be discarded.

Note: This is not a *complete* Proof example, it has been reduced to show
*just* the verify, apply, and sign logic. If you need a working example you should
import ``credits.proof.SingleKeyProof`` from the Common Library and use that.

.. code-block:: python
   :linenos:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from credits.proof import Proof
    from credits.address import CreditsAddressProvider


    class SingleKeyProof(Proof):
        fqdn = 'works.credits.core.SingleKeyProof'
        STATE_NONCE = "works.credits.core.IntegerNonce"

        def __init__(self, address, nonce, challenge, verifying_key=None, signature=None):
            super(SingleKeyProof, self).__init__()

            self.address = address
            self.nonce = nonce
            self.challenge = challenge
            self.verifying_key = verifying_key
            self.signature = signature

        def verify(self, state):
            """
            Verify this proof has been signed and that it's
            signature/verifying_key/challenge is valid against state.

            :returns: result, error
            """
            if (self.signature is None) or (self.verifying_key is None):
                error = "Proof has not been signed."
                self.logger.error(error)
                return None, error

            # Generate an address for this verifying_key, we'll need to validate
            # the key used to sign this proof resolves to a predetermined address.
            nonces = state[self.STATE_NONCE]
            address = CreditsAddressProvider(self.verifying_key.to_string()).get_address()

            if address != self.address:
                error = "Proof for address {} was signed with {}".format(self.address, address)
                self.logger.error(error)
                return None, error

            if not self.verifying_key.verify(self.challenge, self.signature):
                error = "SingleKeyProof failed a signature check against {}".format(address)
                self.logger.error(error)
                return None, error

            known_nonce = nonces[address]
            if self.nonce < known_nonce:
                error = "SingleKeyProof nonce ({}) is less than current nonce ({}) for {}".format(
                    self.nonce,
                    known_nonce,
                    address
                )
                self.logger.error(error)
                return None, error

            return state, None

        def apply(self, state):
            """
            Apply this proof by incrementing the target address' nonce forwards.
            This stops this Proof's parent Transaction from being executed.
            """
            nonces = state[self.STATE_NONCE]
            address = CreditsAddressProvider(self.verifying_key.to_string()).get_address()

            if self.nonce != nonces[address]:
                error = "SingleKeyProof nonce ({}) is not equal to nonce ({}) for {}".format(
                    self.nonce,
                    nonces[address],
                    address
                )
                self.logger.error(error)
                return None, error

            nonces[address] += 1

            return state, None

        def sign(self, signing_key):
            """
            Sign this proof.

            :type signing_key: credits.key.SigningKey
            :rtype: credits.proof.SingleKeyProof
            """
            verifying_key = signing_key.get_verifying_key()
            signature = signing_key.sign(self.challenge)

            return SingleKeyProof(
                address=self.address,
                nonce=self.nonce,
                challenge=self.challenge,
                verifying_key=verifying_key,
                signature=signature,
            )

