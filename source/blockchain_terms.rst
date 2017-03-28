.. _blockchain-terms:

Terminology
^^^^^^^^^^^

Before discussing the details of Credits Core framework architecture it's
worth defining certain basic terms. Some of the terms are so widely used
and abused across the industry it's worth stating so to avoid misconceptions.

.. _blockchain-terms-blockchain:

Blockchain
----------
In the limited technical sense implied here the term `blockchain` referes to
the data structure comprising of cryptographically linked data blocks. It
implies ability to reliably verify the contents of the blocks, but does not
imply distribution of any kind.

The blockchain structure assumes and enforces append-only ability and
intentionally gives no way to edit data cryptographically locked in
the blocks.

Blockchain does not guarantee or prevent the replacement of parts or all
blockchain with other blocks with different contents in case the consensus
mechanics allows that, but that lies with the domain of DLT and consensus
(see next).

.. _blockchain-terms-dlt:

Distributed Ledger Technology
-----------------------------
Distributed Ledger Technology (DLT) is a system that allows reliable
replication of data between miltiple nodes according to certain consensus
algorithm. Usually it assumes full and equal distribution of data between
all nodes and without a central authority, but this really depends on
the concensus algorithm used.

DLT depends on blockchain to operate since without ability to securely and
independently verify the data the DLT degenerates into a simple master-master
replication and can no longer guarantee integrity of data being replicated.

.. _blockchain-terms-consensus:

Consensus
---------
An algorithm defining the way nodes of the DLT agree on the contents of
the next block. Certain algorithms of concensus do allow for replacing
of parts of blockchain starting from a certain block with different versions
of history (forking), while others don't.

Consensus may require a certain arbitrary cryptographic challenge to be
continuously solved (Proof of Work), provide a proof of possession of
certain cryptographic keys (Proof of Stake) or have any other mechanics,
but generally it has to be deterministic and independently verifiable, so
likely will rely on strong cryptography.


Proof of Work consensus
~~~~~~~~~~~~~~~~~~~~~~~

Proof of work is the most often mentioned mechanism for achieving
consensus. Proof of Work requires that a contributor do a deterministically
difficult amount of work that is then easy to check. Bitcoin for example
does this by making miners hash until they get the longest string of zeroes.
This artificially slows down block creation and makes it computationally and
thus financially expensive in the Bitcoin network. Anyone can mine blocks but
given the current normalized difficulty it takes a ridiculously long time
for non-specialized hardware to mine a valid block. This process is the only
way invented by now to reliably implement a public permissionless consensus
where anybody can participate.


Proof of Stake consensus
~~~~~~~~~~~~~~~~~~~~~~~~

Proof of stake is far more like a traditional weighted voting model.
Everyone locks up some value as a promise of their good intentions inside
the system and then there are fixed voting rounds where each person votes
using the weight of the value locked up. In an example both Alice and Bob
stake 50 value into the system, they both have equal votes but neither have
majority. Both together can vote and provide majority for confirming a block.
Anyone can propose a block but only those with stake can vote.

This model does work very well in case the permissioned DLT, i.e. where
participants of consensus have to be given an explicit prior permission to
join. In this case their stake contribution can be verified as part of the
permission granting process.

.. _blockchain-terms-forking:

Blockchain forking
------------------
Applied to blockchain, the event of forking is an occasion where starting
from a certain block the chain of blocks splits into two or more chains,
descending from same parent but further producing different blocks according
to different rules.

Some DLT consensus algorithms allow that, others don't.
