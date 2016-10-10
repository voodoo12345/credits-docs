.. _step-by-step:

Step by step guide
==================


To create a custom permissioned Credits blockchain you will need to go through these steps:

 - create required transforms, proofs and transactions
 - test and verify validity of the code in your local dev environment
 - start your personal blockchain network
 - upload the code to your network and put it into execution
 - create your client application using the same transactions
 - hook your client to the network via node API and start transacting on the blockchain


Create transforms and other source
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Your transforms should implement ``credits.transform.Transform``, see the
:ref:`Transform <transform>` documentation for specifics.

Test and verify your transforms locally
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To test your applications, you can use some helper functions located in ``credits.test``. To test Transforms you can use
``credits.test.check_transform()``. ``check_transform()`` will run a collection of tests against your transform to both
verify it has explicitly met the requirements of ``credits.transform.Transform`` and that all functions execute as 
expected. To see a working example of how to use ``check_transform()`` see: checktransform.py_.

Note that check_transform is not a substitute for standard Unit Testing, you should also perform your own unit testing
of your Transform's ``verify()`` and ``apply()`` functions to check that all potential outcomes are covered.

.. _checktransform.py: https://github.com/CryptoCredits/credits-common/blob/develop/examples/checktransform.py

Get a blockchain network and upload your code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
-------------------
::

    curl -X POST -F "email=test@example.com" -F "password=highlysecurepassword" \
        -F "attributes={}" https://gcloud.credits.works/api/v1/user

Create access token
-------------------
::

    curl -X POST -F "email=test@example.com" -F "password=highlysecurepassword" \
        -F "permissions={}" https://gcloud.credits.works/api/v1/token

In the result of this request you'll need the ``api_key`` field. This will be your access token for
further requests.

Create organisation
-------------------

Organisation ID returned in this response will be needed in further requests. You can retrieve it again
through ``GET /api/v1/user`` endpoint.
::

    curl -X POST --header "Authorization: <your_token>" -F "name=acme-org" \
        -F "attributes={}" https://gcloud.credits.works/api/v1/organization

Patch token
-----------

After creating the organisation you need to patch your token with access to it. By default you would probably want to
add all permissions at once, however in more complex access cases you may have different tokens with specific
access rights configured on each.
::

    curl -X PATCH --header "Authorization: <your_token>" -F "permissions={"<org_id>":{<permissions list>}}" \
        https://gcloud.credits.works/api/v1/token

Create network
--------------

Assuming you have already developed and tested locally your transforms you can provide it to bootstrap your blockchain.
Please notice that module inclusion is a path to file. You need to supply the module contents unescaped and fully
intact including the linebreaks, so it's not possible to include it's contents directly into the ``curl`` call.
::

    curl -X POST --header "Authorization: <your_token>" -F "name=block-network" \
        -F "state=<your_genesis_state>" -F module@<path_to_your_module_file> \
        https://gcloud.credits.works/api/v1/network

Check node names
----------------

Network creation takes some time, and once it's done you'll be able to retrieve node names needed in further queries.
::

    curl -X POST --header "Authorization: <your_token>" \
        https://gcloud.credits.works/api/v1/network/<your_network_id>

Check node status
-----------------

In the node api notice the fact that effectively we're querying the nodes directly, however these calls need to
be proxied through the main API for access control, and thus we need to supply ``/api/v1/node/<your_node_name>`` as
the path to the target node and then ``/api/v1/status`` as the actual method call within that node's API.
::

    curl -X POST --header "Authorization: <your_token>" \
        https://gcloud.credits.works/api/v1/node/<your_node_name>/api/v1/status


