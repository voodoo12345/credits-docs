.. _blockchain:

Credits blockchain 101
======================


Terminology
^^^^^^^^^^^

Before discussing the details of Credits Core framework architecture it's
worth defining certain basic terms. Some of the terms are so widely used
and abused across the industry it's worth stating so to avoid misconceptions.


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


Blockchain forking
------------------
Applied to blockchain forking is an occasion where starting from a certain
block the chain of blocks splits into two chains, descending from same
parent but further producing different blocks according to different rules.

Some DLT consensus algorithms allow that, others don't.


Credits Core blockchain
^^^^^^^^^^^^^^^^^^^^^^^

As any blockchain - the Credits blockchain consists of Blocks. And every
block contains transactions that happened since previous block created.

Unlike other blockchains the Credits blockchain also contains States,
which are snapshots of the contents of the blockchain created after each
block.


.. _blockchain-state:

Blockchain State
----------------

While still being a blockchain, Credits Core blockchain starts with the
state and not a block.

The State, or state of the world, is simply just a key-value map of data.
Credits Core has one global state object, that is comprised of many different
smaller substates. There can be multiple different substates that can
contain any information, all that is important is that every key in the
state is a string. An example of a traditional state as shown:

.. code-block:: json

    {
        "works.credits.loans": {
            "Bob": 200,
            "Dave": 200
        },
        "works.credits.balances": {
            "Alice": 100,
            "Carol": 100
        }
    }

In the example above there is a ``works.credits.balances`` state and a
``works.credits.loans`` state. Each state has to have an FQDN key, and
may have entries inside it. You can define as many states as you need
inside the global state.

Inside the state you can have an arbitrary number of key value pairs.
Usually Credits addresses are used as the keys in a state but this
doesn't always need to be the case, as the keys nature depends on
application's business logic. The state is ordered by insertion order
and the whole state is hashed to provide a state hash.

The initial state of the world may be referenced as Genesis State, but
normally it's called just ``state 0``.

The ``models`` are used to define the internal structure of each substate
and default values for it. Models are discussed in more detail as part
of the :ref:`Dapp <dapp>`.


.. _blockchain-block:

Blockchain Block
----------------

Essentially a block is just a collection of :ref:`transactions <transaction>`.
Blocks are formed by taking valid unconfirmed transactions from the internal
transactions pool and making a block that contains these transactions.
Once the block is formed it is distributed in the network and nodes can decide
to vote and commit to this block. A block also contains information on the
previous state of the world that it is built on. By referencing the previous state,
a node can take the block, check that it is starting in the same
place as the creator of the block, apply the transactions and calculate
the next state. Assuming all nodes are functioning in the same manner -- all
nodes will end up with identical next state. If that is not the case --
network partitioning will occur. This is discussed
:ref:`a little later <dlt-consensus-partitioning>`.


.. _blockchain-structure:

Credits Core blockchain structure
---------------------------------

Building from states and blocks the chain is formed. Because Credits
blockchain has intermediate states it's not a direct link from block to block,
instead, a block is formed from the current state, and then the application of
that block to current state forms the next state.

Imagine starting at the following state 0:

.. code-block:: json

    {
        "works.credits.balances": {
            "Alice": 100,
            "Bob": 0
        }
    }

And there is a transaction that moves 50 credits from ``Alice`` to ``Bob``.
This transaction can apply to state 0, so it is formed into a block that
builds upon state 0.

::

    +-----------+
    |           |
    |  State 0  |
    |           |
    +-----+-----+
          |
          |
    +-----v-----+
    |           |
    |  Block 0  |
    |           |
    +-----------+


The block is then distributed between the nodes and references the state it
is built on. Once the network agrees to make this block the next one in the
chain each node applies transactions in this block to state 0 to produce
the next state.

::

    +-----------+      +-----------+
    |           |      |           |
    |  State 0  |   +-->  State 1  |
    |           |   |  |           |
    +-----+-----+   |  +-----------+
          |         |
          |         |
    +-----v-----+   |
    |           |   |
    |  Block 0  +---+
    |           |
    +-----------+


The new state 1 looks like the following:

.. code-block:: json

    {
        "balance": {
            "Alice": 50,
            "Bob": 50
        }
    }

A new transaction is formed and posted to the blockchain, this transaction
moves the remaining 50 from ``Alice`` to ``Bob``. Another new block is formed
looking like such:
::

    +-----------+      +-----------+
    |           |      |           |
    |  State 0  |   +-->  State 1  |
    |           |   |  |           |
    +-----+-----+   |  +-----+-----+
          |         |        |
          |         |        |
    +-----v-----+   |  +-----v-----+
    |           |   |  |           |
    |  Block 0  +---+  |  Block 1  |
    |           |      |           |
    +-----------+      +-----------+

The process continues and block 1 will be applied to state 1, forming the
next full state.
::

    +-----------+      +-----------+      +-----------+
    |           |      |           |      |           |
    |  State 0  |   +-->  State 1  |   +-->  State 2  |
    |           |   |  |           |   |  |           |
    +-----+-----+   |  +-----+-----+   |  +-----------+
          |         |        |         |
          |         |        |         |
    +-----v-----+   |  +-----v-----+   |
    |           |   |  |           |   |
    |  Block 0  +---+  |  Block 1  +---+
    |           |      |           |
    +-----------+      +-----------+


Leaving it with a final state of:

.. code-block:: json

    {
        "balance": {
            "Alice": 0,
            "Bob": 100
        }
    }

From here onwards other transactions can happen, further mutating global state
and adding new blocks to the chain. The process will run indefinitely as
long as there is a quorum of nodes in the network to agree on blocks and
new valid transactions are coming in.


.. _dlt-consensus:

DLT consensus
-------------

There are many different ways to achieve consensus in DLT. Two of the
common mechanisms that are talked about in blockchain industry are Proof
of Work and Proof of Stake.


Proof of Work
~~~~~~~~~~~~~

Proof of work is the most often mentioned mechanism for achieving
consensus. Proof of Work requires that a contributor do a deterministically
difficult amount of work that is then easy to check. Bitcoin for example
does this by making miners hash until they get the longest
string of zeroes. This artificially slows down block creation and makes it
computationally and thus financially expensive in the Bitcoin network.
Anyone can mine blocks but given the current normalize difficulty it
takes a ridiculously long time for non-specialized hardware to mine a valid
block. This process is the only way invented by now to reliably implement
a public permissionless consensus where anybody can participate.


Proof of Stake
~~~~~~~~~~~~~~

Proof of stake is far more like a traditional weighted election model.
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
 

.. _dlt-consensus-details:

Credits consensus
-----------------

Consensus in Credits is at its heart Proof of Stake. Validators bond value
against as a promise of their honest intentions. Validators attempt to create
valid blocks of unconfirmed transactions. These blocks are distributed between
the validators. Each validator picks a block to vote on (currently this is
the first valid block seen) and then tells the network of their intention to
vote for this block. Once enough votes have been cast for that block to have
a winning concensus everyone announces their intention to commit to that block.
With enough voters committed to a block it becomes ratified history and the
state of the world is upgraded.


.. _dlt-consensus-voting-power:

Voting power
~~~~~~~~~~~~
Voting Power (VP) is fundamental to Credits Core consensus algorithm.
VP is one or more arbitrary numerical values assigned addresses on the
blockchain. These values represent the weight of voting allocated
to each of these addresses. In the simplest case voting power is
distributed equally between addresses attached to each node of the DLT
network.

::

    +-----------+      +-----------+
    |           |      |           |
    |  Node 0   <------>  Node 1   |
    |  VP 100   |      |  VP 100   |
    +-----+-----+      +-+---------+
          |              |
          +---+       +--+
              |       |
            +-v-------v-+
            |           |
            |  Node 2   |
            |  VP 100   |
            +-----------+

In this example each node has VP value of 100 and votes for blocks using
this VP.

A more advanced case:

::

    +-----------+      +-----------+
    |           |      |           |
    |  Node 0   <------>  Node 1   |
    |  VP 100   |      |  VP 100   |
    +-----+-----+      +-+-------+-+
          |              |       |
          +---+       +--+       |
              |       |          |
            +-v-------v-+      +-v---------+
            |           |      |           |
            |  Node 2   <------>  Node 2   |
            |  VP 100   |      |  VP   0   |
            +-----------+      +-----------+

In here three nodes still have same 100 VP, while one other node has
no VP at all. This node cannot vote on the blocks going into the blockchain
but has full visibility of the blockchain contents and can confirm it's
validity.


.. _dlt-consensus-partitioning:

Partitioning
~~~~~~~~~~~~

Forking is not possible in Credits Core, but the Credits network can go into
partitioned state.

Partitioning is a situation where several some nodes disagree with the rest
of the network on the contents of the blockchain or loose connection to the
rest of the network. Forking does not happen in this case because the
minority of the network will never be able to form quorum and progress their
own version of the chain. However those nodes will not progress with the
rest of the network and will effectively stall.

This situation will likely to require manual intervention to identify root
of the problem.
