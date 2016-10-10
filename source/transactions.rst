.. _transactions-transforms-proofs:

Transactions, transforms and proofs
===================================

.. _transaction:

Transaction
^^^^^^^^^^^

Transaction is an :ref:`Applicable <interfaces-applicable>`,
:ref:`Marshallable <interfaces-marshallable>`, and :ref:`Hashable <interfaces-hashable>`
object. Transactions are the units of work used to manipulate state on the blockchain.
Transactions consist of always one ``Transform`` to mutate state and one-or-many ``Proofs`` to authenticate
the changes proposed in the ``Transform``.

To construct a ``Transaction`` it's dependencies should be constructed and injected.
First a ``Transform`` must be constructed, each transform's constructor
will differ based on it's behaviour. The transforms are what will essentially
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

When a Transaction with Transform inside is sent to the Node, it is verified against the current
:ref:`Blockchain State <blockchain-state>`. The ``verify()`` method should perform
checks against state to check if this Transform will be applicable either now or sometime in the
future. If a Transform fails verification at any point it will be flushed from
the Node. Note that if a Transform attempts to modify state during its validation,
changes made to state will be disposed of.

When a Transform is applied it will be given the current state of the Network
and expected to modify and return a new state. During the
:ref:`transaction application<blockchain-applying-transactions>` a Transform may perform
any verification that has to be performed "upon application". If this verification fails,
the apply should fail and return an erroneous result.

Balance Transfer use case
-------------------------

Note: This is not a *complete* Transform example, it has been reduced to show
*just* the verify and apply logic. If you need a working example you should
import ``credits.transform.BalanceTransform`` and use that.

.. code-block:: python
   :linenos:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from credits.transform import Transform


    class BalanceTransform(Transform):
        STATE_BALANCE = "core.credits.balance_app.Balances"

        def __init__(self, from_address, to_address, amount):
            self.from_address = from_address
            self.to_address = to_address
            self.amount = amount

        def verify(self, state):
            """
            Verify it is possible, to apply either now or in the future. Return an
            errornous response if verification fails.
            """
            balances = state[self.STATE_BALANCES]

            if self.from_address not in balances:
                return None, "from_address {} not found in {}.".format(
                    self.from_address,
                    self.STATE_BALANCE
                )

            if self.to_address not in balances:
                return None, "to_address {} not found in {}.".format(
                    self.to_address,
                    self.STATE_BALANCE
                )

            if self.amount <= 0:
                return None, "amount must be greater than 0."

            if balances[self.from_address] < self.amount:
                return None, (
                    "from_address {} does not have the nessasary "
                    "balance (current: {}, required {}) to perform transfer."
                ).format(self.from_address, balances[self.from_address], self.amount)

            return None, None  # Nothing to return, but no error.

        def apply(self, state):
            """
            Modify state, if this fails return an errornous result. Theoretially
            apply should never fail is verify passes.
            """
            balances = state[self.BALANCES]

            try:
                # If the to_address doesn't exist, create it.
                balances[self.to_address] = balances.get(self.to_address, 0) + self.amount
                balances[self.from_address] -= self.amount
                return state, None

            except Exception as e:
                return None, e.message

    ALICE = "1iKEfPKRCXtR5GNGZCi98RuV9ydZuiiYG"
    BOB = "1Jozg7hkLBrjHdf5XECLMipDuYN4bNDxUV"

    STATE = {
        "core.credits.balance_app.Balances": {
            ALICE: 1000,
            BOB: 0,
        }
    }

    TR = BalanceTransform(
        from_address=ALICE,
        to_address=BOB,
        amount=50,
    )

    # Verify TR against the current state.
    # Note that we do nothing with the result as TR.verify() shouldn't return anything.
    result, error = TR.verify(STATE)
    if error is not None:
        raise Exception(error)  # Handle error

    result, error = TR.apply(STATE)
    if error is not None:
        raise Exception(error)  # Handle error

    STATE = result  # result is the STATE with TR applied.


Data storage use case
---------------------

Balance transfer is not the only or the simplest use case, the simplest is probably
the usecase to store events, hashes or metadata on the blockchain. In this case a
following transform can be used:

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
then it fails. When it comes to application then it simply sets the hash against
the time it was applied to the state of the world. This is a far simpler usecase
as there is less input and less validation, but taking this idea a more complex KYC or logging
system could easily be developed.


.. _proof:

Proof
^^^^^

Proof is an :ref:`Applicable <interfaces-applicable>` object requiring both ``verify`` and
``apply`` methods implemented. Proofs are constructed with some sort of resolvable
address, a nonce (which is typically an auto incrementing number), and a
challenge to sign. This challenge will typically be the hash of a Transform.

Once constructed a Proof is *unsigned* and a ``sign`` method must be called
with a ``signing_key`` to generate a ``verifying_key`` and ``signature``. Once
signed a Proof is now considered valid as during it's ``verify`` call it will
attempt to convert the ``verifying_key`` into an address. This address will be
compared to the address the Proof was constructed with.

When Proofs are sent to the Node as a part of Transasction, they are verified
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

