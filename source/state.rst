.. _blockchain-state:

State
=====

State is simply just a key value map of data. Credits has one global state object, this is comprised of many different
smaller states. There can be multiple different states that can contain any information, all that is important is that
the key of the state is a string. An example of a traditional state as shown. 
::

    {
        "loans": {
            "Bob": 200
        },
        "balances": {
            "Alice": 100
        }
    }


In the example above there is a ``balances`` state and a ``loans`` state, these states in the trival example only show one
key in each but it is easy to imagine many different key value pairs. Usually credits addresses are used as the key in a
state but this doesn't always need to be the case. State is ordered by insertion order and is hashed to establish.
