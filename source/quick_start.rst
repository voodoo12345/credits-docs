This tutorial will guide you through the process of preparing your first Credits Blockchain and deploying it to our
cloud-hosted platform.

The Credits PaaS offerings operate by taking three, primary user-supplied components to generate a network definition
and configuration and then deploys and maintains a network based on that configuration on your behalf.

These three components consist of a list of Transforms, an initial genesis state, and any configuration requirements,
such as number of nodes to be deployed.

Transforms
----------

The functionality of your network and how your customers are able to interact with your service is defined based on the
Transforms you enable in your network.

You can either utilize Transforms others have written and released or implement your own Transforms to deploy on your
networks. A basic tutorial of how to implement a basic Credits Transform can be found `here
<%7B%%20post_url%202016-07-31-transform-intro%20%%7D>`__.

Genesis State
-------------

When deploying your network, you need to include the desired initial state of the network. You provide this at the time
of network creation as a JSON string.

On deployment, the network Nodes will automatically consume the JSON input you provide and turn it into a StateView
object that maintains your data in a Merkle Tree structure.

For more information about how StateViews function, a basic tutorial can be found `here
<%7B%%20post_url%202016-07-31-stateview-intro%20%%7D>`__.

Entry Script
------------

When you have decided on the Transforms you wish to enable and have generated your initial Genesis State JSON, we can
now deploy your network live to the Credits Platform.

In order for our system to enable your desired Transform types, you must provide a single python file that includes all
the Transform classes your network requires.

Internally, our platform will then load in all provided Transform types and register them in the DApp's `Component
Registry <%7B%%20post_url%202016-07-31-glossary%%7D#component-registry>`__ for each Node we deploy on your behalf.

For a simple network that has very basic token functionality, you can use `this module file
<../tutorials/examples/simple_value_transform.py>`__ that was created in our `Transforms tutorial
<%7B%%20post_url%202016-07-31-transform-intro%20%%7D>`__.

Deployment via API
------------------

TODO
