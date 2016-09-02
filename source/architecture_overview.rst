This document covers the basic architecture of a node in the current
implementation (core-kt). For more definitions on Credits terms,
please refer to our glossary.

As it currently stands a node is made up of a number of interchangeable
components, each being responsible for acting on messages it is sent.
The diagram below shows the component pieces that make up a canonical
Credits node.

You create a full, running Credits Blockchain Network by deploying a
series of interconnected Credits Nodes which have the same configuration
and together represent at least a Quorum of the Voting Power on the
network.

The DApp
--------

The DApp is the primary container and message broker for a Credits Node.

The DApp registers and holds most of the constituent parts of a Credits
blockchain. Messages are passed into the DApp from the Gateways, and the
DApp orchestrates the consensus mechanism based on these incoming
messages.

|Credits Architecture Diagram|

Additionally, consensus logic is implemented in the DApp directly. To
utilize a different consensus mechanism than our default of a
distributed `Two Phase
Commit <%7B%%20post_url%202016-07-31-glossary%%7D#two-phase-commit>`__
you would implement a separate DApp class which was aware of how to
orchestrate your chosen consensus algorithm.

Gateways
--------

Gateways communicate with the outside world and serve as a translation
layer between external, serialized messages and internal, native
objects.

Gateways live outside the DApp and communicate asynchronously with the
DApp via message queues.

By default, the Credits Framework includes a ZMQ gateway for
communication between peers and an HTTP gateway which provides a RESTful
interface for clients to interact with a running blockchain network.

Functional Logic Components
---------------------------

The following three components are where the business logic lives in a
blockchain network.

Any functionality that happens at the protocol level can be implemented
by extending these three components.

StateViews
~~~~~~~~~~

In the Credits Framework, a StateView acts as a definition of data that
can be stored in a Credits Blockchain.

This definition acts as a schema for defining key -> value relationships
that make up the current state of the world. Generally the key will be a
Credits address and the value will be some form of token or ownership
representation.

StateViews are acted upon by
`Transforms <%7B%%20post_url%202016-07-31-glossary%%7D#transforms>`__
which process the functional logic of a blockchain network.

Transforms
~~~~~~~~~~

A Transform is a defined mechanism for updating a specified
`StateView <%7B%%20post_url%202016-07-31-glossary%%7D#StateView>`__.
Transforms are paired with
`Proofs <%7B%%20post_url%202016-07-31-glossary%%7D#proofs>`__ to make up
a
`Transaction <%7B%%20post_url%202016-07-31-glossary%%7D#transaction>`__.

Each Transform contains a ``verify`` method which verifies whether a
transaction is valid or invalid as applied to a specified StateView, and
an ``apply`` method which updates the provided StateView according to
the payload of the Transform.

Transactions can include time or event driven functionality for smart
contract capabilities.

Proofs
~~~~~~

A Credits Proof is a payload which authorizes a
`Transform <%7B%%20post_url%202016-07-31-glossary%%7D#transforms>`__ on
behalf of an
`Address <%7B%%20post_url%202016-07-31-glossary%%7D#transforms>`__.
Combined, a Proof and a Transform make up the constituent parts of a
Credits
`Transaction <%7B%%20post_url%202016-07-31-glossary%%7D#transaction>`__
which can be processed by the target blockchain network.

There is an enforcement of a one-to-one relationship between a single
Address and a Proof.

For use cases that require multiple parties to authorize an instruction,
you must either create an address using a Proof that allows multiple
parties to be represented by one Address, or you must attach one Proof
for each Address that is required for authorization.

Consensus Components
--------------------

The following six components make up the constituent parts of the
Credits default consensus mechanism.

States
~~~~~~

These are the populated StateViews that make up the current State of the
network.

These are updated and replaced with each Block upgrade whenever the
network finds a new, valid block to be added to history.

Last Block
~~~~~~~~~~

The immediate, previous Block that was applied to generate the current
State of the network.

Transactions Pool
~~~~~~~~~~~~~~~~~

A collection of unconfirmed, but valid Transactions that are available
to be placed in future Proposal Blocks.

Proposals Pool
~~~~~~~~~~~~~~

A collection of unconfirmed Blocks that build upon the current valid
State.

From this group, the Validator Nodes collectively determine which
specific Proposal will become the next valid Block.

Once the consensus process arrives at the next block, all Nodes upgrade
their State with the agreed upon block and restart the consensus process
to find the next block.

Votes Pool
~~~~~~~~~~

Votes are the first phase of the Two Phase Commit process that makes up
the default Credits consensus protocol.

The Votes Pool is a collection of Votes seen by the node during the
consensus process for the current height.

Commits Pool
~~~~~~~~~~~~

Commits are the final phase of the Two Phase Commit process that makes
up the default Credits consensus protocol.

The Commits Pool is a collection of Commits seen by the node during the
consensus process for the current height.

Framework Primitives
--------------------

The complex objects that make up the component pieces of a Credits Node
often heavily rely on various primitive objects that may necessarily be
different when integrating with traditional systems or are required to
be different than our default options for other technical or
non-technical reasons.

To accomodate these requirements, we allow these components to be
provided separately from the business logic that takes advantage of
these primitives so any Credits network can seamlessly incorporate these
changes without completely reimplementing large amounts of
functionality.

Component Registry
~~~~~~~~~~~~~~~~~~

The Component Registry registers all necessary components of a given
Credits blockchain network under each component's FQDN.

When a serialized object is passed to a DApp from a Gateway, the
Component Registry returns a class that can convert the serialized
object into a complex native object that lives in the DApp.

Signature Providers
~~~~~~~~~~~~~~~~~~~

A Signature Provider is a wrapper class for a cryptographic signature
scheme, allowing arbitrary asymmetric signature schemes to be defined
and used in your blockchain network.

Hashing Providers
~~~~~~~~~~~~~~~~~

A Hashing Provider is a wrapper class for a hash function, allowing you
to define and use arbitrary hashing functions in your blockchain
network.

Persistence Providers
~~~~~~~~~~~~~~~~~~~~~

A Persistence Provider works much like a Gateway in that it takes
complex native objects and transforms them into a serialized format.

However, instead of communicating with the outside world, a Persistence
Provider saves the output to a persistence medium such as a database or
disk.

MerkleMap
~~~~~~~~~

The MerkleMap object is an ordered dictionary that maintains a Merkle
Tree of the items it holds.

Address Provider
~~~~~~~~~~~~~~~~

An Address Provider is simply a deterministic method of going from a
String or set of bytes to an Address that can be used as an identity in
a blockchain network.

This is generally a hash function paired with a checksum.

.. |Credits Architecture Diagram| image:: ../img/architecture-overview.png

