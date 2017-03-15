.. _dapp-definition:

Credits Dapp
============

Credits Dapp is a packaged Python module code that provides a
:ref:`state <blockchain-state>` definition, :ref:`transforms <transform>`
that describe how that state is to be mutated and optionally unit tests that
ensure validity of the above.


.. _dapp-models:

Models
------

Credits Dapp Model is a data model used to populate the relevant state.
Model carries an FQDN that defines name for relevant state and defines
defaults that will be used for new keys added to this state.

Model should extended from the base class ``credits.core.model`` and
produce a :ref:`Marshallable <interfaces-marshallable>` object. Here
is an example of a fully valid model:

.. code-block:: python
   :linenos:

    class BalanceModel(Model):
        fqdn = 'works.credits.balance'
        default_factory = int

This model is describing balances and enforces it's values to be integer.


.. _dapp-transforms:

Tranforms
---------

Transform is a standard unit of work in Credits Core. It's used to modify,
transform the state and produce new state. Each transform has to have
``verify()`` and ``apply()`` methods that verify validity of the
transform, it's data and the state it's applied against and apply actual
changes to the state.

Transform has to be an
:ref:`Applicable <interfaces-applicable>`,
:ref:`Marshallable <interfaces-marshallable>` and :ref:`Hashable <interfaces-hashable>`
object. Transforms are constructed with all necessary values required to modify
state during the ``apply()`` method call and are used as a standard unit of work
inside the Credits Core to modify the :ref:`state <blockchain-state>`.

When a Transaction with Transform inside is onboarded to the Node, it is
verified against the current :ref:`Blockchain State <blockchain-state>`.
The ``verify()`` method should perform checks against the state to check
if this Transform will be applicable either now or sometime in the future.
If a Transform fails verification at any point it will be flushed from the
Node. Note that if a Transform attempts to modify state during its
validation, changes made to the state will be disposed of.

When a Transform is applied it will be given the current state
and expected to modify and return a new state. During the
:ref:`transaction application<blockchain-applying-transactions>`, a Transform
may perform any verification that has to be performed "upon application".
If this verification fails, the apply should fail and return an erroneous
result. However, failure of ``apply`` doesn't cause the transaction to be
discarded. It stays in the unconfirmed pool until it's either gets confirmed
or it's ``verify`` method also fails. Only when ``verify`` fails transaction
is discarded and forgotten.


Example transform
~~~~~~~~~~~~~~~~~

Key-Value storage use case is probably the simplest one possible on
the blockchain. In this case following transform can be used:

.. code-block:: python
   :linenos:

    MODEL_KV = "credits.kv.model.kv"

    class KVTransform(transform.Transform):
        fqdn = "credits.kv.transform.KVTransform"
        required_models = {
            MODEL_KV
        }

        def __init__(self, author, key, value):
            self.author = author
            self.key = key
            self.value = value

        @property
        def required_keys(self):
            return {
                self.key,
            }

        @property
        def required_authorizations(self):
            return {
                self.author,
            }

        @classmethod
        def unmarshall(cls, registry, payload):
            return cls(
                author=payload["author"],
                key=payload["key"],
                value=payload["value"],
            )

        def marshall(self):
            return {
                "fqdn": KVTransform.fqdn,
                "author": self.author,
                "key": self.key,
                "value": self.value,
            }

        def verify(self, state):
            """
            Ensure that the value given is in-fact valid JSON.
            """
            json.dumps(self.value)  # failures here are ok

        def apply(self, state):
            """
            Set the value against its key in the Key Value Model.
            """
            state[MODEL_KV][self.key] = self.value # This function mutates state so there is not need to return it


This is an example of fully functional Key-Value transform. It can store
arbitrary values in the blockchain against arbitrary keys. The only
verification done is to make sure the value is JSONifiable.

This and few other trasforms are readily available in
``credits/core/builtin.py`` as built-in transforms.


Reusable dapps
--------------

A Credits Dapp is essentially a Python package, and thus it can be simply
imported and reused as any other regular code library. Several simplest
transforms are available within the Core itself as built-ins, while
several more advances libraries are accessible as additional modules.

Built-ins and third party
~~~~~~~~~~~~~~~~~~~~~~~~~
To use external modules you will need to define them as ``requirements`` in
:ref:`network config <network-architecture>` and also define specific transforms
and models from those modules that you want to be imported into your dapp.

Apart from straight reuse you can also extend and reuse the code from external
modules in your own transforms. To do that just ``import`` it as a regular
python library. You can import from ``credits.core.builtin`` without any
external dependencies. These are available transforms:

 - KVTransform
 - ACLTransform
 - BalanceAdjustTransform
 - BalanceTransferTransform

