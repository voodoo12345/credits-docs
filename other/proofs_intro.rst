Building a Credits Proof Type
=============================

A Credits Proof is a payload which authorizes a `Transform <Broken%20Link>`__ on behalf of an `Address
<Broken%20Link>`__. Combined, a Proof and a Transform make up the constituent parts of a Credits `Transaction
<Broken%20Link>`__ which can be processed by the target blockchain network.

There is an enforcement of a one-to-one relationship between a single Address and a Proof.

For use cases that require multiple parties to authorize an instruction, you must either create an address using a Proof
that allows multiple parties to be represented by one Address, or you must attach one Proof for each Address that is
required for authorization.
