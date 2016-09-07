# Blockchain

Blocks are the building block of the credits blockchain. In simple terms a block is a collection of transactions, that
can be applied to a state to produce a new state.

State zero is the start of the credits blockchain, it details any initial values.
As an example state zero could look like this

```
{
    "balances": {
        "person_a": 100,
        "person_b": 100
    }
}
```

The example shows a balance state which contains two people and each of these people has a starting balance of 100. 
Some transactions are made, these transactions in total move 50 value from person a to person b. A block is formed 
containing these transactions and a reference of the state of the world at that time. The network votes and decides 
on this block and it becomes final. This is how credits forms its blockchain. By linking a state to a block to a state.

Now that the block has been finalised it is applied to state 0 to produce state 1.

This process looks like the following


```
+-----------+   +-----------+
|           |   |           |
|  State 0  |   |  State 1  |
|           |   |           |
+-----+-----+   +-----+-----+
      |               ^      
      |               |      
      |         +-----+-----+
      |         |           |
      +-------->+  Block 0  |
                |           |
                +-----------+
```


The new state 1 looks as follows after the block has been applied.
```
{
    "balances": {
        "person_a": 50,
        "person_b": 150
    }
}
```


This process continues, transactions are now checked against the new state, valid transactions are formed into a block
the block is applied against the current state to produce a new state.

The following diagram shows the process after two blocks have been formed and applied.

```
+-----------+   +-----------+      +-----------+
|           |   |           |      |           |
|  State 0  |   |  State 1  +--+   |  State 2  |
|           |   |           |  |   |           |
+-----+-----+   +-----+-----+  |   +-----+-----+
      |               ^        |         ^
      |               |        |         |
      |         +-----+-----+  |   +-----+-----+
      |         |           |  |   |           |
      +-------->+  Block 0  |  +-->+  Block 1  |
                |           |      |           |
                +-----------+      +-----------+
```
