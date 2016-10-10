.. _proof:

Proof
====

Proof is an :ref:`Applicable <interfaces-applicable>` object requiring both ``verify`` and
``apply`` methods implemented. Proofs are constructed with some sort of resolvable
address, a nonce (which is typically an auto incrementing number), and a
challenge to sign. This challenge will typically be the hash of a Transform.

Once constructed a Proof is *unsigned* and a ``sign`` method must be called
with a ``signing_key`` to generate a ``verifying_key`` and ``signature``. Once
signed a Proof is now considered valid as during it's ``verify`` call it will
attempt to convert the ``verifying_key`` into an address. This address will be
compared to the address the Proof was constructed with.

When Proofs are onboarded on to a Node, they are verified against State to check
that a Signature exists as well as any proof specific ordering is valid. If a
Proof is onboarded in an unsigned state it's parent Transaction will be
discarded.

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

