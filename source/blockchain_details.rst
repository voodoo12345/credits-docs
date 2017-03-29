.. _blockchain_details:


Credits Core
^^^^^^^^^^^^

As any blockchain - the Credits blockchain consists of Blocks. And every
block contains transactions that happened since the previous block created.

Unlike other blockchains, the Credits blockchain also contains States,
which are snapshots of the contents of the blockchain created after each
block is appended to the blockchain.


.. _blockchain-state:

Blockchain State
----------------

While still being a blockchain, Credits Core blockchain starts with the
state and not a block.

The State, or state of the world, is simply just a key-value map of data.
Credits Core has one global state object, that is comprised of many different
:ref:`models <dapp-models>`. The Dapp models are effectively data models
and are used to define the data structure that is supposed to go into specific
substate, and also provide an FQDN that will be used to reference that part of
the state.

There can be multiple different models in the global state, and it can contain
arbitrary values, all that is important is that every key in the state is a
string. An example of a traditional state as shown:

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

In the example above there is a ``works.credits.balances`` model and a
``works.credits.loans`` model. Each model has to have an FQDN key and
may have entries inside it. You can define as many models as you need
inside the global state.

Inside the model you can have an arbitrary number of key-value pairs.
Usually, Core addresses are used as the keys in a model but this
doesn't always need to be the case, as the keys nature depends on
application's business logic. The model is ordered by insertion order
and the whole global state is hashed to provide a state hash.

The initial state of the world may be referenced as Genesis State, but
normally it's called just ``state 0``.


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
the next state. Assuming all nodes are following the same logic -- all
nodes will compute identical next state. If that is not the case --
network partitioning will occur, and this edge case is discussed
:ref:`a little later <dlt-consensus-partitioning>`.


.. _blockchain-structure:

Blockchain structure
--------------------

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


.. _dlt-consensus-details:

Credits Core consensus
----------------------

Credits Core consensus is a leaderless two-phase commit algorithm with
variable :ref:`voting power <dlt-consensus-voting-power>`.

This means that each and every DLT network participant is equal in its
rights to gather transactions from the unconfirmed transactions pool and
form a block, and they are free to vote on the blocks that make the most
sense according to current block validation rules. Also, votes may have
different weights though according to voting power distribution for a given
network.

Essentially Credits Core consensus is a variant of Proof of Stake algorithm.

.. _dlt-consensus-example:

Consensus example sequence
~~~~~~~~~~~~~~~~~~~~~~~~~~

Assume a network of three nodes, A B and C. Network starts at height 0, no
blocks exist yet and the current state of the network is state 0.

#. Node A receives a valid transaction from a client through HTTP gateway.
#. Node A verifies the transaction and onboards it, adding it to the
   unconfirmed transactions pool.
#. Node A recognises new transaction in the pool and tries to form a block
#. Node A forms a block proposal and sends it out to other nodes in the
   network.
#. Nodes B and C receive the block proposal and verify the proposed block
   for validity.
#. Nodes B and C confirm block validity and start voting on the block.
   This is the process of `voting`, phase one of the consensus algorithm.
   At this point only one block proposal exists, so all votes are given to
   this block.
#. Votes are exchanged and if one block reaches the quorum of
   :ref:`Voting Power <dlt-consensus-voting-power>` backing -- it is now a
   `voted block`. The phase one is done.
#. When the node receives enough votes on a block and thus finds out that the
   block was `voted`, the node goes into phase two of the consensus algorithm
   -- `committing` to the block.
#. Once the selected block has received quorum Voting Power backing - it is
   considered committed to be the next block in the chain. Phase two is done.
#. Every node receiving the block with enough committing VP attached to it
   adds this block to the chain and persists it on whatever storage is used
   with that particular node.
#. The process can start over from scratch if there are new transactions
   in the pool.

In a more complex real life situations, there will be multiple transactions
forming block proposals on different nodes and nodes will exchange and
vote on proposals until one of the proposals reaches the quorum.

The consensus algorithm depends on several things:

a. There has to be a required quorum defined. The current default is at
   least 51% of all Voting Power present in the system.
b. There has to be voting power available in the system, and whoever is
   executing the votes using it -- has to have access to the corresponding
   private keys. This is discussed in more details in
   :ref:`Voting <dlt-consensus-voting-power>` section.
c. There has to be a connection between nodes that will allow to propagate
   the blocks and votes.

Given these conditions are met - the consensus algorithm will function
correctly.


.. _dlt-consensus-voting-power:

Voting and Voting Power
~~~~~~~~~~~~~~~~~~~~~~~
Voting in Credits Core consensus is strongly tied to Voting Power (VP).
VP is one or more arbitrary integer values assigned addresses on the
blockchain. These values are stored in the blockchain
:ref:`State <blockchain-state>` and represent the weight of votes allocated
to each of these addresses.

.. code-block:: json

    {
        "credits.voting.model.voting": {
            "19joM8wBG7bAmBypMj23DBmmjGmqfxL4Bj": 100,
            "14RixTSeLtit5GJoDKdEJ23ob74vcvcFvv": 100
        }
    }

Example above is a subset of the blockchain state showing two addresses with
``100`` of voting power assigned to each. The VP figure itself assigned to
each address is of little importance, what matters is the relative weight
of VP of each address to the total VP declared.

To use this voting power, i.e. cast a vote one must have in possession the
private key corresponding to the address VP is assigned to to be able
to prove the ownership. By default, every node of the Credits DLT network
holds a private key to one of the VP addresses and casts votes with it,
and VP itself is split equally between all parties.

However it is totally possible practically to create new addresses and
assign VP to them, imbalance the system by giving some addresses more VP
than others or strip out VP completely from some addresses
(set to ``0`` or delete address).

In the edge case when VP is completely absent, so the VP model contains
no addresses with none-zero VP values -- DLT will stall and won't be able
to progress, i.e. confirm new blocks.


So in the simplest default case VP is distributed equally between addresses
attached to each node of the DLT network.

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

In this example each node has VP value of ``100`` and votes for blocks using
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

In here three nodes still have same ``100`` VP, while one other node has
no VP at all. This node cannot vote on the blocks going into the blockchain
but has full visibility of the blockchain contents and can confirm it's
validity.


.. _dlt-consensus-partitioning:

Partitioning
~~~~~~~~~~~~

Forking is not possible in Credits Core, but the Credits network can go into
the partitioned state.

Partitioning is a situation where the quorum cannot be reached in a part of
the network because either the network connectivity is impaired and nodes
cannot propagate the votes on new blocks or the nodes cannot agree on the
rules of the network and do not cast votes on blocks that they assume invalid.

In both of these scenarios the Voting Power required to choose and commit to
new block becomes unavailable to part or the whole network, and the affected
part of the network stalls, i.e. cannot continue to grow the chain.

Since not enough VP is available to progress the chain and vote on blocks
-- no minority chain will form, the forking or chain reorganisation will
not happen at any circumstance and any data that was voted and committed
to the chain before the partitioning will be not affected.

In this case, if only a minority of the network's VP is affected and the
majority is still both in agreement and has enough VP available to
form a quorum -- the majority of the network will continue to operate normally.

The partitioning situation will likely to require manual intervention to
identify and address the root of the problem, whether it is a connectivity
issue, consensus disagreement or anything else.
