.. _blockchain:

.. _blockchain-general:

Blockchain
==========

Building from state and blocks a chain can be established. Because credits has intermediate states its not a direct link
from block to block, instead a block is formed from a state, and then the application of that block forms a new state.

Imagine starting at state 0.

.. code-block:: json

    {
        "balance": {
            "Alice": 100,
            "Bob": 0
        }
    }


And there is a transaction that moves 50 credits from ``Alice`` to ``Bob``. 

This transaction can apply to state 0, so it is formed into a block that builds upon state 0.
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


The block is distributed between the nodes and references the state it is built on, once the network agrees to make this
block each node applies this block to state 0 to produce the next state.
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


The new state 1 looks like the following

.. code-block:: json

    {
        "balance": {	
            "Alice": 50,
            "Bob": 50
        }
    }

A new transaction is formed, this transaction moves the remaining 50 from ``Alice`` to ``Bob``. Another new block is
formed looking like such
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


Leaving it with a final state of

.. code-block:: json

    {
        "balance": {
            "Alice": 0,
            "Bob": 100
        }
    }

