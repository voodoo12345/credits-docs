.. _getting-started:

Getting started
===============

To start building a distributed ledger, you will need to install Credits Core
framework, lay out your dapp scaffold and bootstrap your development network.

The Credits Core framework is written in Python and going forwards we will be
using standard Python tools to work with it.


Installation
^^^^^^^^^^^^

Before installing the framework please make sure these external system
dependencies are met:
 - Python3.5+
 - python pip
 - libffi
 - libzmq

To install the framework run:

.. code-block:: bash

    pip install git+ssh://git@github.com/CryptoCredits/credits-core.git

You may want to use `virtualenv <https://docs.python.org/3/library/venv.html>`_
to contain all installed python packages instead of putting it into your
global system space.

The framework contains ``credits`` CLI tool that you will use to template
your dapps and bootstrap your local dev network.


Dapp scaffold
^^^^^^^^^^^^^
Once Credits Core is installed you can run:

.. code-block:: bash

    credits dapp create --name <dapp_name>

This will create a ``./<dapp_name>`` folder and a scaffold of the Credits
dapp inside it. You will use it to develop your distributed ledger.

Your dapp is effectively a python package, and it needs to be made
discoverable for Credits Core framework to allow it to be later included
into the ledger. To achieve this run:

.. code-block:: bash

    pip install -e ./<dapp_name>

The ``network.yaml`` file is created as part of the dapp scaffold. It is
used in both the dapp configuration, and as a base for the next step --
network bootstrap.


Network bootstrap
^^^^^^^^^^^^^^^^^
You need a distributed ledger network to both run your dapp and integrate
your client applications. To bootstrap a distributed ledger network run:

.. code-block:: bash

    credits local create ./<dapp_name>/network.yaml

This will roll out configuration for a single node network in the
``./node00`` folder. It contains all you will need to run
the Credits Core network node.

You can also create multi-node network if needed by adding param
``--count <number_of_nodes>``.


Network run
^^^^^^^^^^^
To run the network node you will need to start the nodes you have just
created. You do this with:

.. code-block:: bash

    credits local run ./node00/*.yaml

To start the node you need to provide it with both the network config,
which is copied to the node folder and the node config itself. Both
are located in the node folder as ``network.yaml`` and ``node.yaml``.

This starts node process in the foreground and outputs the node logs
into stdout.

By default ``loglevel`` in node config is set to ``WARNING``, you may want
to make it more verbose with ``--log-level <LOG_LEVEL>`` option.

If you need to run a network of several nodes - you will need to run each node
separately.


