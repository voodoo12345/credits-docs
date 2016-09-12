# Block

In the simplest terms a block is just a collection of transactions.  Blocks are formed by taking unconfirmed
transactions that are valid and making a block that contains these transactions.  Once the block is formed it is
distributed in the network and nodes can decide to vote and commit to this block. A block also contains information of
the state of the world that it is built on. By referencing the state, a node can take the block, check that it is
starting in the same place as the creator of the block and then apply the transactions. 

Applying a block is the process of applying each transaction in order. Each transaction will produce a new state once it
is applied, by applying every transaction in the block this will form the next state of the world. 
