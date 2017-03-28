.. _interfaces:

FAQ
===

Is Credits a cryptocurrency?
----------------------------
No. Credits is not a cryptocurrency and has nothing to do with
cryptocurrencies. Credits is a software framework for creation of
applications based on Distributed Ledger Technology (DLT).

Is Credits a distributed ledger or blockchain?
----------------------------------------------
The **distributed ledger** vs **blockchain** comparison is politically
charged question so is not easy to answer in the common sense. However,
within this documentation we are using these terms in strictly technical
sense, and technical definitions of both are given in
:ref:`terms <_blockchain-terms>` page.

In the strict technical sense Credits Core is a DLT framework.

Is there a public Credits token?
--------------------------------
No. Since Credits is not a cryptocurrency there is no underpinning token
or anything of a kind.

Is there a public Credits' blockchain?
--------------------------------------
No. Credits is not a public permissionless blockchain in the sense of e.g.
Bitcoin being called a public blockchain, and is not supposed to be.
Credits Core is a DLT framework designed to help developers of conventional
end user applications do their work, i.e. build applications, without having
to worry much about implications of a distributed ledger.

A public but permissioned distributed ledger can be built with Credits, and
this distinction is discussed later in this FAQ.

Is there a Credits blockchain at all?
-------------------------------------
Yes, and more than one. But those blockchains are mostly private, so it is
not possible to access them without explicit permission from their
respective owners.

So what is Credits Core?
------------------------
Credits Core is a software framework for building DLT based applications.
It is a developer tool designed to help developers build applications
using Distributed Ledger Technology. It has an underlying blockchain data
structure in strict technical definition, but is not meant to be a public
ledger like Bitcoin or Ethereum. It is designed primarily for building
applications with private permissioned blockchains.

What is a private blockchain?
-----------------------------
Private blockchain is quite simply - a blockchain that is kept privately
and not exposed to the world. Very much like a traditional database - the
private blockchain is a blockchain-as-database, backend system accessed by
the backend application developers, devops and other engineers, but not
accessible to the public.

Private blockchain is by definition permissioned since once needs an
explicit permission to access such blockchain. However not being private
doesn't make a blockchain immediatelly permissionless.

What is permissioned blockchain or distributed ledger?
------------------------------------------------------
Permissioned blockchain is a blockchain where one needs to obtain a
permission to join a specific distributed ledger network and participate in
it's consensus.

This may be a private ledger, where one needs explicit permission to even
see the contents of specific blockchain, or a public permissioned ledger
where the blockchain contents is available for reading without prior
permission, but joining the network is still possible only after
explicit permission from the distributed ledger owners/operators.

A permissioned ledger may be public, where the contents of the blockchain
is available for reading and verification, or private.

Is Credits a Proof of Work or Proof of Stake?
---------------------------------------------
Credits is a Proof of Stake system. For more details please refer to
refer to :ref:`Credits Core consensus algorithm <dlt-consensus-details>`
description.

Is Proof of Stake actually useful?
----------------------------------
In a permissioned distributed ledger there is no need to impose an arbitrary
artificially complicated task (proof of work) to ensure the alignment
of participants' economic incentives with the goals of the network's
existence. A permissioned system by definition is created by certain
participants with a specific goal and is not meant to be used by the
uninterested public. Thus the participants of the network are supposed to
have an interest in network's existence and functioning through interests
external to the network itself.

Stakes in a permissioned system are a mere reflection of stakeholders input
into the creation and maintenance of the said system. Stakes can and should
reflect the relationships between the stakeholders outside of the network
and correspond to mutual duties and obligations.

In this sense Proof of Stake consensus algorithm is a much better reflection
of the permissioned distributed ledger system then a Proof of Work.

What about "nothing at stake" problem?
--------------------------------------
Nothing-at-stake is a conceptual problem and a possible attack vector
in a Proof of Stake distributed ledgers that implies that uninterested
operators that may hold "stake" in the Proof of Stake consensus not
necessarily have actual stake in the said system, i.e. have nothing to
lose if system is malfunctioning or subverted.

This is a problem in a permissionless Proof of Stake systems since wide
public is allowed to join the concensus without an obligation to have
explicit stake in the system. This is not so much of a problem in the
permissioned Proof of Stake ledgers because all participants of the network
are expected to have an external interest in network's existence and good
standing. So in the permissioned system the participants are expected to
have stake prior to joining the network, otherwise they should have no reasons
and should be not allowed to participate, and so the nothing-at-stake problem
does not apply.

What about blockchain forking?
------------------------------
Forking is not possible in Credits consensus algorithm. For more details
please refer to :ref:`consensus algorithm details <dlt-consensus-details>`.
