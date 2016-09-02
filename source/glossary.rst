When building nascent technology, terms are often used in ill-defined or conflicting ways. The definitions below
represent the way we use these terms in Credits.

Address
-------

A truncated, base-58 encoded hash plus a checksum that represents a lookup key for state on a given blockchain.

In general, an address lives as a reference to an identity on a blockchain network. Transactions are authorized by an
Address, generally sending value to another Address.

Addresses can represent a single public key, a static group of public keys, or a dynamic membership group assembled
under a Namespace.

Apply
-----

One of the three functions necessary to define a Transform or Proof Type.

Defines the change resulting from a valid Transform or Proof being Processed on a provided StateView.

Block
-----

A list of ordered Transactions, built off a base StateView, which are valid transactions according the rules of the
network.

When applied to a StateView, the Transactions are applied in order, returning an updated StateView reflecting the
changes detailed in the individual Transactions.

Blockchain
----------

A cryptographically-verifiable shared State, maintained by a distributed set of Validators.

A Blockchain network consists of a group of nodes collectively maintaining this State according to a predefined set of
rules for structuring data and authorizing updates.

Credits
-------

A built-from-scratch framework for creating purpose-specific, Interoperable Blockchains.

Also refers to various PaaS Platforms built on the Credits Core Framework.

Crosschain
----------

A Credits Blockchain that utilizes information or functionality maintained in a separate Blockchain network using SPV
proofs or Multi-Signature Addresses.

The Credits answer to Interoperability.

Cryptocurrency
--------------

A pseudonymous, censorship-resistant form of Virtual Cash that uses a blockchain to maintain accounts and process
transactions.

Generally utilizes a token native to that blockchain network.

Commit
------

The second half of a two-phase commit when When voting rounds have chosen a winning Proposal from a group of candidate
Proposals.

Component Registry
------------------

The Component Registry registers all necessary components of a given Credits blockchain network under each component's
FQDN.

When a serialized object is passed to a DApp from a Gateway, the Component Registry returns a class that can convert the
serialized object into a complex native object that lives in the DApp.

Count
-----

When used in reference to Blocks, Transactions, or StateView, this number represents the unique count of these items
that exist in a given node in a network, discounting any pruned or discarded objects.

For example, a node that syncs up to a Blockchain with Height 200, but prunes the first 100 blocks in history would have
a Block Count of 100.

This number is specific to a given Node.

DApp
----

Short for *Decentralized Application*.

The primary container and messaging broker for a Credits Node.

ed25519
-------

a digital signature scheme using a variant of Schnorr signature based on Twisted Edwards curves. It is designed to be
faster than existing digital signature schemes without sacrificing security.

It was developed by a team including Daniel J. Bernstein, Niels Duif, Tanja Lange, Peter Schwabe, and Bo-Yin Yang.

Genesis StateView
-----------------

The Genesis StateView is the initial starting point for a blockchain network.

This StateView is then built on by proposed, then accepted, blocks, keeping the entire Blockchain Network in sync.

Height
------

When used in reference to Blocks, Transactions, or StateViews, this number represents the unique count of these items
that exist in the full, unpruned blockchain history in a given network.

For example, a blockchain with a Block Height of 1 means there has only been one Block in existence since the Genesis
StateView.

This Number should be identical between all fully-synced Nodes.

Interoperability
----------------

The ability to transport value, information, or identity from one blockchain network to another using SPV proofs and
multi-signature transactions.

This can refer to several different specific types of interoperability:

-  The ability to trigger changes in State in one blockchain as a result of events that have happened in another
   blockchain.
-  The ability to have one blockchain act as an autonomous user of a second blockchain network.
-  The ability to process transactions that atomically process across multiple blockchain networks with no possibility
   of a transaction being successfully committed to one blockchain and not the other(s).  

Merged Mining
-------------

A mechanism in which Proof of Work miners can be paid by multiple networks for finding valid blocks.

A key component for Bitcoin denominated Sidechains.

MerkleMap
---------

The MerkleMap object is an ordered dictionary that maintains a Merkle Tree of the items it holds.

Multi-Key Addresses
-------------------

Any Address that is controlled by 2 or more Public Keys.

Static Multi-Key Addresses are made up of a fixed set of participants, while Namespace addresses can add or remove
arbitrary participants over time.

Either type of Multi-Key Address can insert additional data into the
Address String creation process in order to form multiple addresses with
the same membership set without prior coordination.

Namespace
---------

A Namespace is a dynamic set of participants who are authorized to collectively act on behalf of a Namespace.

The purpose of the Namespace is two-fold. The first is the ability to claim a human-readable identifier as a stand-in
for a standard Address.  The second is to allow for dynamic Multi-Key Addresses.

A Transaction, similar to a Bond or Unbond Transaction for Validators, allows members to Join or Leave Namespaces.

Node
----

A logical Peer in a Blockchain Network. This Peer can either be a Validator or a User on the Network.

A blockchain network is made up of one or more Nodes.

Nonce
-----

An integer that demarcates an ordering of Transactions authorized by a given address. Nonces must be used in order.

Nonces are stored and incremented globally for any given Dapp.

Processed
---------

When used in reference to Blocks, Transactions, or StateViews, this number represents the unique count of these items
processed by a given Node since it has existed.

This number is specific to a given Node.

Proofs
------

A combination of Signatures, Public Keys, and Meta-data that signifies the entity or entities controlling an Address
have authorized an action detailed in the corresponding Transaction.

Proofs can be arbitrarily added to any Credits Network, depending on what types of authorizations the Network would like
to support.

Proposal
--------

A proposed block as a candidate for the next block in a blockchain.

Proposals are voted on in voting rounds by Validators, until the winning Proposal is chosen by at least a quorum of the
Validator Network in a Two Phase Commit process.

Post-Block Processing
---------------------

An optional function making up part of the definition of a Transaction
Type.

If a Transaction needs to have an action be performed at a later date, time or event driven, this behaviour can be
defined as part of a Post-Block Processing hook that is performed at the end of Processing any Block, applying any
necessary changes if a time or event trigger has been met as of that Block confirmation.

Sidechain
---------

A project undertaken by Blockstream, Inc to allow Bitcoin to be used in Merge-Mined Blockchains that may have additional
features than Bitcoin Core.

Merge-Mined Bitcoins on these networks are then held in escrow by the Bitcoin miners.

Signature
---------

A String or ByteArray proving a specified Public Key signed a provided message.

The message is most often a Transaction from Users or a Vote or Commit from Validators.

Simplified Protocol Verification (SPV)
--------------------------------------

This mode of syncing to a Blockchain Network allows a user to only process a fraction of Blocks and Transactions while
still gaining most of the benefits of full validation.

This mode is currently incompatible with being a Validator Node.

Smart Contracts
---------------

A way of programmatically defining ways to interact with State inside a Blockchain that can optionally include logic
either when a Transaction is submitted or autonomously at a future point in time.

This is achieved out of the box with a Credits Transaction.

State
-----

The current state of a Blockchain Network.

A mapping of Addresses to value held, Transactions Processed, and history saved.

StateView
---------

The name for a given snapshot of State in any given Blockchain Network.

Also refers to the implementation of expressing this state in the Credits Core Framework.

Static Multi-Key Address
------------------------

An Address that refers to a set of m-of-n public keys where M valid signatures must be present on any Transaction
performing an action on behalf of the Address.

Transaction
-----------

A structured set of data that represents an action being authorized on behalf of a network participant.

Once a Transaction is Processed, the requisite change in State is processed by Nodes and reflected in the current
StateView.

A Transaction is made up of a Transform and one or more Proofs that authorize that Transform.

Transforms
----------

A functor that can be verified against, and applied to, the state of a blockchain network.

Defines functionality and interactions users can make within a system.

Combined with one or more Proofs to make up a Transaction.

Two Phase Commit
----------------

In transaction processing, databases, and computer networking, the two-phase commit protocol (2PC) is a type of atomic
commitment protocol (ACP). It is a distributed algorithm that coordinates all the processes that participate in a
distributed atomic transaction on whether to commit or abort (roll back) the transaction (it is a specialized type of
consensus protocol). The protocol achieves its goal even in many cases of temporary system failure (involving either
process, network node, communication, etc. failures), and is thus widely used.

In a "normal execution" of any single distributed transaction, i.e., when no failure occurs, which is typically the most
frequent situation, the protocol consists of two phases:

1) The commit-request phase (or voting phase), in which a coordinator process attempts to prepare all the transaction's
participating processes (named participants, cohorts, or workers) to take the necessary steps for either committing or
aborting the transaction and to vote, either "Yes": commit (if the transaction participant's local portion execution has
ended properly), or "No": abort (if a problem has been detected with the local portion).

2) The commit phase, in which, based on voting of the cohorts, the coordinator decides whether to commit (only if all
have voted "Yes") or abort the transaction (otherwise), and notifies the result to all the cohorts. The cohorts then
follow with the needed actions (commit or abort) with their local transactional resources (also called recoverable
resources; e.g., database data) and their respective portions in the transaction's other output (if applicable).

The Credits Protocol additionally operates without the requirement for a coordinator, as the decision to commit comes as
a result of quorum being reached during the voting phase with all participants upgrading once they have reached a quorum
consensus on the commit phase.

Validator Network
-----------------

A list of addresses that are authorized to collectively order and timestamp incoming Transactions to a Blockchain
network.

These are optionally weighted per participant, which is useful in situation where Votes refer to a value in the outside
world, such as shares in a company or a surety bond.

Validator
---------

A participant in a Validator Network.

Verify
------

One of the methods necessary to define a Transform or Proof Type.

Defines what constitutes a valid Transform or Proof of this type, including who is authorized to submit the associated
payload and what data is expected to be provided.

Vote
----

A Signature made by a Validator during a Voting Round on a candidate
Proposal.

First part of the Two Phase Commit process.

Voting Round
------------

A time-boxed window where Validators propagate Votes for a candidate Proposal.

At the end of each Voting Round, the Validators reset their list of Votes sent and received.

If a quorum of Validators on a single Proposal is reached before the end of a Voting Round, participating Validators
race to affix a Finalizing Signature to the Proposal, permanently confirming it as part of the history of the
Blockchain.
