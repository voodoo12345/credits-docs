In order to bootstrap a network into existence, you need to provide an initial state which defines network participants
and their role in the network.

This state includes an initially populated `StateView <%7B%%20post_url%202016-07-31-glossary%%7D#stateview>`__ for each
StateView type your network enables, even if a specific StateView is initially empty.

When deploying a self-hosted network, you will need to specify the initial set of
`Validators <%7B%%20post_url%202016-07-31-glossary%%7D#validator>`__ whereas they will be automatically populated and
maintained for you when deploying to a Credits Platform which is cloud-hosted.

For example, a simple genesis state for deploying to a Credits Platform
might look like the following:

.. code:: json

    {
      "credits.works.core.balances" : {
        "Address1" : 500,
        "Address2" : 1250,
        "Address3" : 3725
      },
      "credits.works.core.admins" : {
        "AddressA" : "active",
        "AddressB" : "inactive"
      }
    }

However, if we were deploying to a self-hosted offering, you would additionally have to include the full initial set of
Validators.

.. code:: json

    {
      "credits.works.core.VotingPower" : {
        "Validator1" : 100,
        "Validator2" : 100,
        "Validator3" : 100,
        "Validator4" : 100,
        "Validator5" : 100
      },
      "credits.works.core.balances" : {
        "Address1" : 500,
        "Address2" : 1250,
        "Address3" : 3725
      },
      "credits.works.core.admins" : {
        "AddressA" : "active",
        "AddressB" : "inactive"
      }
    }

A network you generate on-premises with this genesis state will then utilize ``Validator1`` through ``Validator5`` as
it's set of initial validators, as well as deploying with the functional state represented by FQDNs
``credits.works.core.balances`` and ``credits.works.core.admins``.
