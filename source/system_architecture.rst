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

Overview
^^^^^^^^

This is the overview of possible architectures of an end user application built on top of the Credits blockchain.

The blockchain application architectures can be viewed from two angles - the deployment basing angle and the
node ownership angle.

Deployment options
^^^^^^^^^^^^^^^^^^

You may deploy your private blockchain in the Credits PaaS cloud system, within your own infrastructure or as a mixture
of both. PaaS deployments are the simplest way and are most suitable for early stage development and simpler
deployments that do not need tight control over the infastructure. Own infrastructure deployments are suitable for
higher profile production systems where full ownership of the system is required.

Also own infrastructure deployments are required if you're looking to develop a more customized system on our
framework that goes beyond of what is possible within just Transactions and Transforms. E.g. if you need to redefine
the consensus mechanics or introduce own network transport protocols.

A hybrid deployment where parts of the blockchain system are deployed on-premises at own infrastructure, while
other parts are hosted at Credits PaaS cloud might be useful in case of multiownership setup, where both ownership
control and cost effectiveness are critical.

Ownership options
^^^^^^^^^^^^^^^^^

When talking about ownership we always mean control over blockchain nodes.
In case of single ownership the


Cloud single owner blockchain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Simplest scenario is when the blockchain is hosted on the public PaaS cloud we're offering. In this case you don't
need to worry about the hosting and support of your blockchain backend and only build and deliver your application to
your clients.

The archtecture will look roughly like this:
::

    +-----------------------------------------------------+
    | Credits PaaS                                        |
    |                                                     |
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
                  HTTP REST          HTTP RESTful
                     API                API
                      |                  |
         +------------v-----+      +-----v------------+
         | Client app       |      | Client app       |
         |                  |      |                  |
         |                  |      |                  |
         |    +------------+|......|    +------------+|
         |    | blockchain ||      |    | blockchain ||
         |    | client     ||      |    | client     ||
         |    +------------+|      |    +------------+|
         +------------------+      +------------------+

So in this architecture your client app will connect to the blockchain via HTTP REST API exposed by the nodes.
You will need to have a blockchain client to form transactions and sign them, it may be hosted inside your web app's
backend or compiled into your mobile app. The easiest way to implement it is to use our
:ref:`common library<common-library>` written in Python. It will take care of all needed cryptography for you
and will only require you to define needed business logic.

This architecture will suit the simpler case where you require the immutability of the blockchain and the audit
trail, but do not need a crossverification of the data between multiple parties, as essentially the whole blockchain
is controlled by single party in this case.

Also under this scenario may also fall the case when the whole blockchain is hosted on your ownm infrastructure,
still under full control of one party.


Mixed hosting blockchain
^^^^^^^^^^^^^^^^^^^^^^^^

More advanced scenario

::

    +-----------------------------------------------------+
    | Credits PaaS                                        |
    |                                                     |
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
                  HTTP REST          HTTP RESTful
                     API                API
                      |                  |
         +------------v-----+      +-----v------------+
         | Client app       |      | Client app       |
         |                  |      |                  |
         |                  |      |                  |
         |    +------------+|      |    +------------+|
         |    | blockchain ||      |    | blockchain ||
         |    | client     ||      |    | client     ||
         |    +------------+|      |    +------------+|
         +------------------+      +------------------+

