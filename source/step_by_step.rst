.. _step-by-step:

Step by step guide
==================


To create a custom permissioned blockchain with Credits framework you will
need to go through these steps:

1. create required transforms, proofs and transactions
2. test and verify the validity of the code in your local dev environment
3. start your private blockchain network and upload the code
4. create your client application using the same transactions
5. hook your client to the network via node API and start transacting on the blockchain


.. _step-by-step-create-transform:

1. Create transforms and other modules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the simplest scenario, you will need to only implement your transforms. You
can use ``SingleKeyProof`` provided in the Common library as a default way to
prove transaction validity with one signing key per signature. And your
transaction in the simplest scenario can also be the default ``Transaction``
provided in the common library.

So in this simplest case, you may have just one transform. The transform must implement
``credits.transform.Transform``. You can find more details on implementing
transforms and actual examples in :ref:`Transform <transform>` documentation.


.. _step-by-step-test-verify:

2. Test and verify your transforms locally
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before uploading your modules to the network you might want to verify it
locally to confirm that created code conforms to required interfaces.

To do this you can use helper functions located in ``credits.test``. To test
Transforms you can use ``credits.test.check_transform()``.

``check_transform()`` will run a collection of tests against your transform
to both verify it has explicitly met the requirements of
``credits.transform.Transform`` interface and that all functions execute as
expected. Here is an example of how to use ``check_transform()``:

.. code-block:: python
   :linenos:

    # Construct the Transform with these.
    kwargs = {
        "addr_from": "alice_address",
        "addr_to": "bob_address",
        "amount": 100,
    }

    # This is an initial state that both verify and apply will use
    state = {
        "credits.test.Balances": {
            "alice_address": 1000,
        }
    }

    # This is how state should look after the Transform has applied to it.
    state_expected = {
        "credits.test.Balances": {
            "alice_address": 900,
            "bob_address": 100,  # This key is added as a result of transform.apply()
        }
    }

    # This call should succeed without assertion errors or exceptions
    test.check_transform(
        cls=BalanceTransform,
        dependencies=[],
        kwargs=kwargs,
        state=state,
        state_expected=state_expected,
    )


Note that ``check_transform`` is not a substitute for standard Unit Testing,
you should also perform standard unit testing of your Transform's ``verify()``
and ``apply()`` methods to check that all potential outcomes are covered.

You can find a full example of Transform creation and testing in check_transform.py_.

.. _check_transform.py: https://github.com/CryptoCredits/credits-common/blob/develop/examples/check_transform.py


.. _step-by-step-get-network-upload:

3. Get a blockchain network and upload your code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once your modules are written and tested locally - it's time to deploy a test
blockchain network and see it in action. The easiest way to do this is to use
our public PaaS, which is at the moment available for free. You can register
via the REST API and get a running network in few HTTP requests.

If you're working for a government agency and looking to use our GCloud PaaS API - the process is
essentially same, except that your PaaS account will be disabled by default until we'll get you
through the formal onboarding process.

The most complicated option would be to get Credits framework running on your own infrastructure.
This is technically possible and not that complex since we ship it in handy prebuilt Docker containers,
but since at the moment Credits Core is a proprietary software - you will have to go through sales channel
first and purchase a license before we'll be able to hand the software to you.
Also, the PaaS registration and network bootstrap guide will not apply in this case.

Below are the API call steps needed to register with public PaaS and create 
a test blockchain network.

Register an account
-------------------

.. code-block:: bash

    curl -X POST -F "email=test@example.com" -F "password=highlysecurepassword" \
        -F "attributes={}" https://public.credits.works/api/v1/user

Create access token
-------------------

.. code-block:: bash

    curl -X POST -F "email=test@example.com" -F "password=highlysecurepassword" \
        -F "permissions={}" https://public.credits.works/api/v1/token

You will need to save the ``api_key`` returned the response to this request. This
will be your access token for further requests.

Create organisation
-------------------

Organisation ID returned in this response will be needed in further requests.
You can save it now or retrieve again later through ``GET /api/v1/user`` endpoint.

.. code-block:: bash

    curl -X POST --header "Authorization: <your_token>" -F "name=acme-org" \
        -F "attributes={}" https://public.credits.works/api/v1/organization

Patch token
-----------

After creating the organisation you need to patch your token with rights
definitions to be able to access it. By default you would probably want to
add all permissions at once, however, in more complex access cases you may
have different tokens with specific access rights configured on each.
See full permissions list in the :ref:`Paas API<paas-api>`.

.. code-block:: bash

    curl -X PATCH --header "Authorization: <your_token>" -F "permissions={"<org_id>":{<permissions list>}}" \
        https://public.credits.works/api/v1/token

Create network
--------------

Assuming you have already developed and tested locally your transforms
you can now provide it to bootstrap your custom blockchain. Please notice
that module inclusion is a path to a local file. For this parameter you need to
supply the contents of the module created earlier at steps 1 and 2.
The "module" is essentially a correct and complete Python source file
with your Transforms and Proofs. An example of fully implemented BalanceTransform
can be found in balance_transform.py_.

You need to supply module source code fully intact including the line breaks
to preserve the validity of the Python source, so it's not possible to include
it's contents directly into the ``curl`` call string, and it has to be
included as part of the multipart POST request.

State can be initialized with the following format

.. code-block:: javascript
    
    {
        "works.credits.balances": {
            "default": 0,
            "values": {
                "bob": 100,
                "alice": 400
            }    
        }
    }

Default defines what is returned when you use safe access on a state and the key is not present. This can be any valid JSON value.
Values is the initial state of the world, in the example above bob will have 100, alice will have 400 and any other key will return
the default 0. You can define as many states via the FQDN as you want but each state must have a defined default and values.

.. code-block:: bash

    curl -X POST --header "Authorization: <your_token>" -F "name=block-network" \
        -F "state=<your_genesis_state>" -F module@<path_to_your_module_file> \
        https://public.credits.works/api/v1/network

.. _balance_transform.py: https://github.com/CryptoCredits/credits-common/blob/develop/examples/balance_transform.py

Check node names
----------------

Network creation takes some time, and once it's done you'll be able to retrieve
node names needed in further queries.

.. code-block:: bash

    curl -X POST --header "Authorization: <your_token>" \
        https://public.credits.works/api/v1/network/<your_network_id>

Check node status
-----------------

In the node api notice the fact that effectively we're querying the nodes
directly, however these calls need to be proxied through the main API for
access control purposes, and thus we need to supply
``/api/v1/node/<your_node_name>`` as the path to the target node and
then ``/api/v1/status`` as the actual method call within that node's API.

.. code-block:: bash

    curl -X POST --header "Authorization: <your_token>" \
        https://public.credits.works/api/v1/node/<your_node_name>/api/v1/status


.. _step-by-step-create-client:

4. Create client application
^^^^^^^^^^^^^^^^^^^^^^^^^

Once your network is up and running - you can create the client side application
for it. Essentially you will need to use the same modules that were uploaded to
the network, but incorporate it into the client side application.

Of course, the bulk of your clientside application is something we cannot
define, it may be a web system, a mobile app, an IoT device etc.
However, the general requirements will be that it has to be able to
persistently store client's keys, and will conform to the
Transforms and Proofs interfaces uploaded into the blockchain.

As an example here is the simple Python script that implements
generating user's keys, dumping those to disk (persistence), creating valid
Transaction and sending it to the node's URL provided.


.. code-block:: python
    :linenos:  

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    import requests
    from credits.key import ED25519SigningKey
    from credits.address import CreditsAddressProvider
    from credits.proof import SingleKeyProof
    from credits.transaction import Transaction

    # create a key for Alice using default key provider
    alice_key = ED25519SigningKey.new()

    # create Alice's address using default address provider
    alice_address = CreditsAddressProvider(alice_key.to_string()).get_address()

    # Saving the key to disk by marshalling it
    with open("alice_key.json", "w") as out:
        out.write(alice_key.marshall()

    # Loading it would be also simple when you'll need it
    # with open("alice_key.json") as keyfile:
    #    payload = json.load(keyfile)
    #    alice_key = ED25519SigningKey.unmarshall(None, payload)

    # create transform to send credits from Alice to Bob
    transform = BalanceTransform(amount=100, addr_from=alice_address, addr_to="bob_address")

    # sign the needed proof with Alice' key
    proof = SingleKeyProof(alice_address, 1, transform.get_challenge()).sign(alice_key)

    # form a transaction
    transaction = Transaction(transform, {alice_address: proof})

    # POST your transaction to the node in your network
    requests.post("https://public.credits.works/api/v1/node/<your_node_name>/api/v1/transaction", 
        headers={"Authorization": "<your_api_key>"},        
        data={"transaction": json.dumps(transaction.marshall())}
    )


You can also find this example in the sample_client.py_.

.. _sample_client.py: https://github.com/CryptoCredits/credits-common/blob/develop/examples/sample_client.py


.. _step-by-step-connect-and-start:

5. Connect client application to the blockchain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once the application is written and deployed you can start transacting
on the blockchain. If everything is done correctly in the previous steps
- nothing blockchain specific is needed at this level.


