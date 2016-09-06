# Transaction
Transactions are the unit of work used to maniupulate state on the blockchain.
They provide both a ```Transform``` to mutate state and one-or-many ```Proof```s
to authenticate the changes proposed in the ```Transform```.

To construct a ```Transaction``` it's dependencies should be constructed and injected.
First a ```Transform``` must be constructed, each ```Transform```s constructor
will differ based on it's behaviour, see your ```Transform```'s constructor for
more information.

Once the ```Transform``` has been created, create create at least one Proof
using the ```Transform.hash()``` method to generate a challenge for the Proof.
Note that a ```Transform``` has a ```required_authorizations``` method which
will return a list of mandatory addresses that must provide a Proof.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from credits.transform import BalanceTransform
from credits.proof import SingleKeyProof
from credits.key import ED25519SigningKey
from credits.address import CreditsAddress
from credits.hash import SHA256HashProvider

HASH_PROVIDER = SHA256HashProvider()

DAVID_SK = ED25519SigningKey.new()
DAVID_VK = DAVID_SK.get_verifying_key()
DAVID_ADDR = CreditsAddress(DAVID_VK.to_string()).get_address()

JACOB_SK = ED25519SigningKey.new()
JACOB_VK = JACOB_SK.get_verifying_key()
JACOB_ADDR = CreditsAddress(JACOB_VK.to_string()).get_address()

transform = BalanceTransform(DAVID_ADDR, JACOB_ADDR, 100)
# transform.required_authorizations() ==  [DAVID_ADDR]

transaction = Transaction(
    transform=transform,
    proofs=[
        SingleKeyProof(DAVID_ADDR, 0, transform.hash(HASH_PROVIDER)).sign(DAVID),
    ]
)

transaction.marshall()  # This output can be jsonified and POSTed to /api/v1/transaction
```
