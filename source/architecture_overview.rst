.. _architecture-overview:

Application architectures
=========================

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

