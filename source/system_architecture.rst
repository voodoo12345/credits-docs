.. _system-architecture:

System architecture
===================

The overall system of the custom blockchain application consists of the blockchain
network that is formed from blockchain nodes and client applications that connect
to the blockchain nodes via HTTP API.


.. _architecture-blockchain-network:

Blockchain network
^^^^^^^^^^^^^^^^^^
Blockchain network is a set of multiple blockchain nodes. Each node is connected to
other nodes, and in total enough nodes should be online to form a quorum needed to
elect new blocks according to Credits blockchain default
:ref:`consensus mechanics <blockchain-consensus>`.


.. _architecture-blockchain-node:

Blockchain node
^^^^^^^^^^^^^^^

Blockchain node is essentially a continuously running server that is connected to
every other server inside blockchain network. The blockchain node software is the
Credits Framework plus the :ref:`Transactions <transaction>`, :ref:`Transforms <transform>`
and :ref:`Proofs <proof>` implemented and provided by the owner of the private blockchain.
It may be either uploaded via PaaS API or placed onto the blockchain node hosts directly
if you're hosting your own system.

The node holds it's own copy of the blockchain, participates in blocks voting and block confirmations
according to the current consensus mechanics and serves the HTTP API to the blockchain clients.

.. _architecture-blockchain-client:

Blockchain client
^^^^^^^^^^^^^^^^^

Blockchain client is an application that connects to blockchain node HTTP API and
interacts with the blockchain on one side and with the user on the other.

The client is expected to be able to generate and store user's private keys, create
transactions, provide proofs and upload signed transactions to the blockchain via HTTP API.
Also it should be able to interact with end user, however design on this side is beyond the
scope of Credits blockchain development.

For all the cryptographic and blockchain primitives implementation and for reference interfaces
we advise to use open source Python :ref:`common libary <common-library>` provided by Credits.
However it's totally possible to create separate implementation in another language as long as
the interfaces are conformed.


How this all fits together
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

So how do we put together all pieces of the system and deploy a production
architecture fit for specific purpose?

The blockchain application architectures can be viewed from two angles -
the deployment angle and the node ownership angle.

Deployments
-----------

You may deploy your private blockchain in the Credits PaaS cloud system, within your
own infrastructure or as a mixture of both. PaaS deployments are the simplest way and
are most suitable for early stage development and simpler deployments that do not need
tight control over the infastructure. Own infrastructure deployments are suitable for
higher profile production systems where full ownership of the system is required.

Also own infrastructure deployments are required if you're looking to develop a more
customized system on our framework that goes beyond of what is possible within just
Transactions definitions. E.g. if you need to redefine the consensus mechanics or
introduce own network transport protocols.

A hybrid deployment where parts of the blockchain system are deployed on-premises
at own infrastructure, while other parts are hosted at Credits PaaS cloud might be
useful in case of multiparty setup, where both node control and cost
effectiveness are critical.

Ownerships
----------

When talking about ownership we always mean control over blockchain nodes.
And of course since Credits blockchain is only meant to be used in the permissioned
environment - we assume all parties having access to blockchain are explicitly allowed
to do so and no new parties can join the network without prior permission.

In case of single ownership all of the nodes in the network are controlled by single
party, and no other parties are expected. This is the simplest use case, and while
it doesn't utilise some of the key blockchain benefits like external audit, it may still
be a practical use case for the sake of immutability or record of history.

Multiple ownership system is a preferred blockchain usecase, as it allows to utilise
external auditability features and establish trust between the parties.

Multiple ownership systems may be formed of equal or roughly equal parties, where
every party has equal rights in consensus voting, or it can be formed with unequal
proportion of voting power where one or few parties are controlling the blockchain
while others are allowed to connect to network, read and verify the blockchain,
but have no say over what will be written to it.

Architecture scenarios
^^^^^^^^^^^^^^^^^^^^^^

Below we'll cover a few common scenarios practical for different use cases.

Cloud based single owner blockchain
-----------------------------------

Simplest scenario is when the blockchain is hosted on the Credits' public PaaS.
In this case you don't need to worry about the hosting and support of your blockchain
backend and only build and deliver your application to your clients.

The architecture will look roughly like this:
::

    +-----------------------------------------------------+
    | Credits PaaS                                        |
    | Alice                                               |
    |   +------------+  +------------+  +------------+    |
    |   | blockchain <--+ blockchain <--+ blockchain |    |
    |   |    node    +-->    node    +-->    node    |    |
    |   +---------^--+  +--^-------^-+  +--^---------+    |
    |             |        |       |       |              |
    |           +-v--------v-+  +--v-------v-+            |
    |           | blockchain <--+ blockchain |            |
    |           |    node    +-->    node    |            |
    |           +-----+------+  +--------+---+            |
    |                 |                  |                |
    +-----------------+------------------+----------------+
                      |                  |
                  HTTP REST          HTTP REST
                     API                API
                      |                  |
         +------------v-----+      +-----v------------+
         | Client app       |      | Client app       |
         | Bob              |      | Carol            |
         |    +------------+|......|    +------------+|
         |    | blockchain ||      |    | blockchain ||
         |    | client     ||      |    | client     ||
         |    +------------+|      |    +------------+|
         +------------------+      +------------------+

So in this architecture your client app will connect to the blockchain via HTTP REST API
exposed by the nodes. You will need to have a blockchain client to form transactions and
sign them, it may be hosted inside your web app's serverside backend or compiled into your mobile app.
The easiest way to implement it is to use our :ref:`common library<common-library>` written
in Python. It will take care of all needed cryptography for you and will only require you to
define needed business logic.

This architecture will suit the simpler case where you require the immutability
of the blockchain and the history trail, but do not need a crossverification
of the data between multiple parties. Client access may be given to other allowed
parties beyond the blockchain owner himself.

Also under this scenario may also fall the case when the whole blockchain is
hosted on your own infrastructure, still under full control of one party.


Multiparty cloud-based blockchain
---------------------------------

A more advanced scenario, where multiple parties own pieces of the system and
control their own nodes, however still having the nodes being hosted in the cloud.

::

    +----------------------------------+    +----------------------------------+
    | Credits PaaS                     |    | Credits PaaS                     |
    | Alice                            |    | Bob                              |
    |   +------------+  +------------+ |    |   +------------+  +------------+ |
    |   | blockchain <--+ blockchain <-+----+---+ blockchain <--+ blockchain | |
    |   |    node    +-->    node    +-+----+--->    node    +-->    node    | |
    |   +---------^--+  +--^---------+ |    |   +---------^--+  +--^---------+ |
    |             |        |           |    |             |        |           |
    |           +-v--------v-+         |    |           +-v--------v-+         |
    |           | blockchain |         |    |           | blockchain |         |
    |           |    node    |         |    |           |    node    |         |
    |           +-----+------+         |    |           +-----+------+         |
    |                 |                |    |                 |                |
    +-----------------+----------------+    +-----------------+----------------+
                      |                                       |
                  HTTP REST                               HTTP REST
                     API                                     API
                      |                                       |
         +------------v-----+                       +---------v--------+
         | Client app       |                       | Client app       |
         | Alice            |                       | Bob              |
         |    +------------+|                       |    +------------+|
         |    | blockchain ||                       |    | blockchain ||
         |    | client     ||                       |    | client     ||
         |    +------------+|                       |    +------------+|
         +------------------+                       +------------------+

In this scenario two parties (Alice and Bob) own equal parts of the blockchain
and each has a client that connects to their own parts of the system. The
client apps for Alice and Bob may be identical, i.e. two deployments of the
same app, or completely different apps developed separately. The only requirement
is that these two apps have to be compatible in the blockchain client
implementation.


Multiparty mixed hosting blockchain
-----------------------------------

An advanced scenario with multiple parties using the blockchain and one ultimate
controlling party holding authority over the additions to it.

::

    +----------------------------------+    +------------------+    +------------------+
    | Credits PaaS                     |    | own deployment   |    | own deployment   |
    | Alice                            |    | Bob              |    | Carol            |
    |   +------------+  +------------+ |    | +------------+   |    | +------------+   |
    |   | blockchain <--+ blockchain <-+----+-+ blockchain <---+----+-+ blockchain |   |
    |   |    node    +-->    node    +-+----+->    node    +---+----+->    node    |   |
    |   +---------^--+  +--^---------+ |    | +---------^--+   |    | +---------^--+   |
    |             |        |           |    |           |      |    |           |      |
    |           +-v--------v-+         |    +-----------v------+    +-----------v------+
    |           | blockchain |         |                |                       |
    |           |    node    |         |                |                       |
    |           +-----+------+         |                |                       |
    |                 |                |                |                       |
    +-----------------+----------------+                |                       |
                      |                                 |                       |
                  HTTP REST                         HTTP REST               HTTP REST
                     API                               API                     API
                      |                                 |                       |
         +------------v-----+               +-----------v------+    +-----------v------+
         | Client app       |               | Client app       |    | Client app       |
         | Alice            |               | Bob              |    | Carol            |
         |    +------------+|               |    +------------+|    |    +------------+|
         |    | blockchain ||               |    | blockchain ||    |    | blockchain ||
         |    | client     ||               |    | client     ||    |    | client     ||
         |    +------------+|               |    +------------+|    |    +------------+|
         +------------------+               +------------------+    +------------------+

In this scenario Alice has three nodes deployed with Credits PaaS, while Bob
and Carol have one node each deployed at their premises. Bob and Carol have same
access to blockchain as Alice, however because they together have two nodes,
while Alice has three - Alice nodes will always win the block voting process,
so Bob and Carol effectively have no say over the writing to the blockchain,
but can utilise it contents and verify it's validity.

Practically there is no need to control voting via pure number of nodes, concensus
mechanics allows customised voting weight distributions that would result in the
same behaviour without a need to host an overwhelming amount of physical blockchain
nodes.


