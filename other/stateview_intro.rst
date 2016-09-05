StateViews are a declaration of data. This is the core concept behind a Credits Distributed Ledger. This is the data on
which the transforms operate.

StateViews are represented by a unique FQDN.

In the `transform <./transform-intro.md>`__ tutorial an example of a token value transfer is implemented. This document
demonstrates how to define any corresponding StateViews a Transform may require.

While a StateView is represented as a `MerkleMap <%7B%%20post_url%202016-07-31-glossary%%7D#merklemap>`__ when loaded
into a DApp, we define an initial genesis State using JSON which is then loaded into the DApp when it is being
constructed and used to populate a MerkleMap internally.

.. code:: json

    {
        "works.credits.value": {
            "16Asg9qfvHTCJEGzzWkpUTvVwAhNZHAzcQ": 100
        }
    }

We simply define a map where the key is the state's FQDN and our populated state as the value. It's common to use a
mapping of Credits Addresses to value as the state.

A StateView can contain any combination of four basic primitive types: Strings, Integers, Lists, and Dictionaries
containing any of these four types as shown below.

.. code:: json

    {
        "works.credits.staff": {
            "16Asg9qfvHTCJEGzzWkpUTvVwAhNZHAzcQ": {
                "name" : "John Smith",
                "age" : 33,
                "status" : "Full time staff",
                "extra-data" : {
                    "miscellaneous" : "Test Information",
                    "two" : 3
                }
            }
        }
    }

A developer can define many state views at the same time by simply using different FQDNs. Internally, the DApp will
automatically generate a MerkleMap populated with the provided data and register it in it's ``states`` object for future
utilization.

.. code:: json

    {
        "works.credits.admins": ["16Asg9qfvHTCJEGzzWkpUTvVwAhNZHAzcQ"],
        "works.credits.stock_values": {
            "GOOG": 1043
        }
    }

It is down to the implementer to state the default value logic. In some systems it might be required that all values are
defined clearly in state before being used, in others its acceptable for a default to be used in place of no value.

Genesis State
~~~~~~~~~~~~~

When bootstrapping a network, we need to seed it with initial state in order to know the initial set of participants and
parameters of the network. To do so, the participants launching the network collectively agree on this initial state,
and it becomes part of the definition of the network as a result.

Blocks that conform to the rules of the network than modify this state as the Validator Network comes to consensus on
valid Transactions that are submitted by the users of the network.

Enforced Schema
~~~~~~~~~~~~~~~

In the near future, we will allow validation of StateView entries and enforcement of restrictions on StateView contents.

This will make it more straightforward to enforce correctness, especially as the usage of third-party Transforms becomes
more prevalent.

When released, this capability will be announced on our mailing list and on our Knowledge Base.
