.. _step-by-step:

Step by step guide
====


To create a custom permissioned Credits blockchain you will need to do this:

 - get yourself a blockchain network
 - create required transforms, proofs, transactions and other code
 - test and verify vailidity of the code in your local dev environment
 - upload the code to your network and let it execute
 - create your client application using the same transactions
 - hook your client to the network via node API and start transacting on the blockchain

Get a blockchain network
^^^^

The easiest way to do this is to use our public PaaS, which is at the moment avaialable for free.
You can register via the REST API and get a running network in few HTTP requests.

If you're working for a government agency and looking to use our GCloud PaaS API - the process is
essentially same, except that your PaaS account will be disabled by default until we'll get you
through the formal onboarding process.

The most complicated option would be to get Credits framework running on your own infrastructure.
This is technically possible and not that complex, since we ship it in handy prebuilt Docker containers,
but since at the moment Credits Core is a proprietary software - you will have to go through sales channel
first and purchase license before we'll be able to hand the software to you.
Also the PaaS registration and network bootstrap guide will not apply in this case.

PaaS selfservice network creation steps.

Register an account
----





