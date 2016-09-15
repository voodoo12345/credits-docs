# Transform
Transforms are an [Applicable](interfaces.html#applicable),
[Marshallable](interfaces.html#marshallable), and
[Hashable](interfaces.html#hashable) object.  They are constructed with all
nessasary values required to modify state during the ```apply``` method and are
used as the standard unit of work inside the framework to modify State.

When a Transform is onboarded onto an Node, it is verified against the current
Blockchain State. The verify method should perform checks against state to
check if this Transform will be applicable either now or sometime in the
future. If a Transform fails verification at any point it will be flushed from
the Node. Note: If a Transform attempts to modify state during its validation,
changes made to state will be disposed of.

When a Transform is applied it will be given the current state of the Network
and expected to modify and return a new state. during the apply a Transform may
perform any verification that has to be performed "upon application". If this
verification fails, the apply should fail and return an errornous result
result.

Note: This is not a _complete_ Transform example, it has been reduced to show
_just_ the verify and apply logic. If you need a working example you should
import ```credits.transform.BalanceTransform``` and use that.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from credits.transform import Transform


class BalanceTransform(Transform):
    STATE_BALANCE = "core.credits.balance_app.Balances"

    def __init__(self, from_address, to_address, amount):
        self.from_address = from_address
        self.to_address = to_address
        self.amount = amount

    def verify(self, state):
        """
        Verify it is possible, to apply either now or in the future. Return an
        errornous response if verification fails.
        """
        balances = state[self.STATE_BALANCES]

        if self.from_address not in balances:
            return None, "from_address {} not found in {}.".format(
                self.from_address,
                self.STATE_BALANCE
            )

        if self.to_address not in balances:
            return None, "to_address {} not found in {}.".format(
                self.to_address,
                self.STATE_BALANCE
            )

        if self.amount <= 0:
            return None, "amount must be greater than 0."

        if balances[self.from_address] < self.amount:
            return None, (
                "from_address {} does not have the nessasary "
                "balance (current: {}, required {}) to perform transfer."
            ).format(self.from_address, balances[self.from_address], self.amount)

        return None, None  # Nothing to return, but no error.

    def apply(self, state):
        """
        Modify state, if this fails return an errornous result. Theoretially
        apply should never fail is verify passes.
        """
        balances = state[self.BALANCES]

        try:
            # If the to_address doesn't exist, create it.
            balances[self.to_address] = balances.get(self.to_address, 0) + self.amount
            balances[self.from_address] -= self.amount
            return state, None

        except Exception as e:
            return None, e.message

DAVID = "1iKEfPKRCXtR5GNGZCi98RuV9ydZuiiYG"
JACOB = "1Jozg7hkLBrjHdf5XECLMipDuYN4bNDxUV"

STATE = {
    "core.credits.balance_app.Balances": {
        DAVID: 1000,
        JACOB: 0,
    }
}

TR = BalanceTransform(
    from_address=DAVID,
    to_address=JACOB,
    amount=50,
)

# Verify TR against the current state.
# Note that we do nothing with the result as TR.verify() shouldn't return anything.
result, error = TR.verify(STATE)
if error is not None:
    raise Exception(error)  # Handle error

result, error = TR.apply(STATE)
if error is not None:
    raise Exception(error)  # Handle error

STATE = result  # result is the STATE with TR applied.
```

# Other usecases

Sometimes a balance transfer is not what is needed, instead the usecase is to store events, hashes or metadata on the 
blockchain. In this case a simpler transform can be used

```
class LogHashTransform(Transform):
    STATE_BALANCE = "core.credits.log.hashes"

    def __init__(self, hash):
        self.hash = hash

    def verify(self, state):
        if state[self.LOG_STATE][self.hash]:
            return None, "Already have this hash logged!"
        return None, None 

    def apply(self, state):
        state[self.LOG_STATE][self.hash] = {"logged_at": time.asctime()}
        return state, None
```

This transform will first verify the hash is not already loaded. If it is loaded then it fails. When it comes to 
application then it simply sets the hash against the time it was applied to the state of the world.

This is a far simpler usecase as there is less input and less validation, but taking this idea a more complex KYC or logging
system could easily be developed.
