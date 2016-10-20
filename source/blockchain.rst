.. _blockchain:

Blockchain
==========

The Credits Blockchain Framework as an application is a distributed software service that runs on
multiple servers called :ref:`Nodes <architecture-blockchain-node>`. Every node holds a copy
of the Credits blockchain as a data structure. This data structure consists of cryptographically
connected :ref:`Blocks <blockchain-block>` and :ref:`States <blockchain-state>`. New blocks and
states are added to the blockchain by :ref:`applying transactions <blockchain-applying-transactions>`
and electing the best blocks through node :ref:`consensus <blockchain-consensus>`.

.. _blockchain-state:

Blockchain State
^^^^^^^^^^^^^^^^

The State is simply just a key-value map of data. Credits Framework has one global state object,
this is comprised of many different smaller states. There can be multiple different
substates that can contain any information, all that is important is that every key in the
state is a string. An example of a traditional state as shown:

.. code-block:: json

    {
        "works.credits.loans": {
            "default": 0,
            "values": {
                "Bob": 200,
                "Dave": 200
            }
        },
        "works.credits.balances": {
            "default": 0,
            "values": {
                "Alice": 100,
                "Carol": 100
            }
        }
    }

In the example above there is a ``works.credits.balances`` state and a
``works.credits.loans`` state. Each state has to have an FQDN key, and
must have defined ``default`` and ``values`` entries. You can define
as many states as you need inside the global state.

Inside the state ``values`` you can have an arbitrary number of key value
pairs. Usually credits addresses are used as the keys in a state but this
doesn't always need to be the case, as the keys nature depends on application's
business logic. The state is ordered by insertion order and the whole
state is hashed to provide a state hash.

The ``default`` inside the State defines what is returned when you use safe
access on a state and the key is not present in the ``values``. Default
can be any valid JSON value. In the example above a query on any other key
apart from the explicitly defined ones will return the default integer 0.


.. _blockchain-block:

Blockchain Block
^^^^^^^^^^^^^^^^

In the simplest terms, a block is just a collection of :ref:`transactions <transaction>`.
Blocks are formed by taking valid unconfirmed transactions from the internal transactions
pool and making a block that contains these transactions. Once the block is formed
it is distributed in the network and nodes can decide to vote and commit to this block.
A block also contains information on the state of the world that it is built on. By
referencing the state, a node can take the block, check that it is starting in the same
place as the creator of the block and then apply the transactions.


.. _blockchain-onboarding-transactions:

Onboarding transactions
^^^^^^^^^^^^^^^^^^^^^^^

Onboarding transaction, transform or proof means generally to get it accepted into the
node's unconfirmed transactions pool. In case of PaaS deployment, the onboarding happens
by sending the marshalled transaction to the node HTTP API. In other more complex
deployments the HTTP API gateway can be replaced with other transport, e.g. TCP socket,
CLI interface, RPC interface etc. Onboarding here serves as a transport agnostic term
for the act of accepting transaction into the node.


.. _blockchain-applying-transactions:

Applying transactions
^^^^^^^^^^^^^^^^^^^^^

Applying a transaction means to execute the :ref:`Transform <transform>` contained
inside transaction against the current global state and get the next global state.
Consider same global state as above:

.. code-block:: json

    {
        "loans": {
            "Bob": 200,
            "Dave": 200
        },
        "balances": {
            "Alice": 100,
            "Carol": 100
        }
    }

If we apply transaction that moves 50 credits from Alice to Bob. Then the next global state will be:

.. code-block:: json

    {
        "loans": {
            "Bob": 250,
            "Dave": 200
        },
        "balances": {
            "Alice": 50,
            "Carol": 100
        }
    }

This will reflect the fact that Alice has loaned further 50 credits to Bob.

Applying a block is the process of applying each transaction in order. Each transaction
will produce a new state once it is applied, and by applying every transaction in the block
this will form the next state of the world after the block.

Any :ref:`Applicable <interfaces-applicable>` object should be able to apply itself.


.. _blockchain-consensus:

Blockchain consensus
^^^^^^^^^^^^^^^^^^^^

There are many different Consensus mechanisms. Two of the common mechanisms that are talked about in
blockchain are Proof of Work and Proof of Stake.  


Proof of Work
-------------
Proof of work is the more commonly talked about mechanism for achieving consensus. Proof of work 
requires that a contributor do a deterministically difficult amount of work that is then easy to check. 
Bitcoin does this by making miners hash until they get the longest string of zeroes this 
artificially slows down block creation in the bitcoin network. 
Anyone can mine blocks but given the current normalize difficulty it takes a long time for 
non-specialized hardware to mine a valid block. Think of this as like a lottery,
everyone is turning a crank and one person is rewarded every x minutes.


Proof of Stake
--------------

Proof of stake is far more like a traditional election model. Everyone locks up some value as a promise of their 
good intentions inside the system and then there are fixed voting rounds where each person votes using the weight of the value locked 
up. In an example both Alice and Bob stake 50 value into the system, they both have equal votes but neither have
majority. Both together can vote and provide majority for confirming a block. Anyone can propose a block but only
those with stake can vote.


Credits consensus
-----------------

Consensus in Credits is at its heart Proof of Stake. Validators bond value against as a promise of their honest intentions.
Validators attempt to create valid blocks of unconfirmed transactions. These blocks are distributed between the validators.
Each validator picks a block to vote on (currently this is the first valid block seen) and then tells the network of their
intention to vote for this block. Once enough votes have been cast for that block to have a winning concensus everyone announces
their intention to commit to that block. With enough voters committed to a block it becomes ratified history and the state of the 
world is upgraded.

.. _blockchain-structure:

Blockchain structure
^^^^^^^^^^^^^^^^^^^^

Building from states and blocks the chain can be created. Because Credits blockchain
has intermediate states it's not a direct link from block to block, instead, a
block is formed from the current state, and then the application of that block to
current state forms the next state.

Imagine starting at the following state 0:

.. code-block:: json

    {
        "balance": {
            "Alice": 100,
            "Bob": 0
        }
    }

And there is a transaction that moves 50 credits from ``Alice`` to ``Bob``. This
transaction can apply to state 0, so it is formed into a block that builds upon state 0.
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


The block is then distributed between the nodes and references the state it is
built on. Once the network agrees to make this block the next one in the chain
each node applies this block to state 0 to produce the next state.
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
moves the remaining 50 from ``Alice`` to ``Bob``. Another new block is
formed looking like such:
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

The process continues and block 1 will be applied to state 1, forming the next full state. 
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
long as there is a quorum of nodes in the network and new valid transactions
are coming in.
