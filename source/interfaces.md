# Interfaces

## Marshallable

The Marshallable interface provides the nessasary methods to convert fully
realised instances into more primative maps and to reverse the process by
converting these maps back into instances.

### FQDN
Marshallable objects must contain a fully qualified domain name, this is used by
the framework to resolve objects from serialised representations. The FQDN
should be made as an attribute of the implementor as either a class level
variable or a property, either are acceptable. This FQDN will be used during the
unmarshalling process to locate an associated Class to instanciate.

```python
class Foo:
    FQDN = "fully.qualified.domain.name.Foo"

# OR

class Bar:
    @property
    def FQDN(self):
        return "fully.qualified.domain.name.Bar"
```

### marshall
Marshalling is the process of converting a instance into a map of primative
variables which can be then serialized. This allows libraries like ```json``` and
```msgpack``` to use their ```dumps``` methods to convert the output of
```marshall``` to a string or series of bytes suitable for transport over the
network. The ```fqdn``` used by the framework to resolve the implementor's class
and convert the output of ```marshallable``` back into an instance. Note that a
Marshallable object should also marshall any of it's subcomponents.

```python
def marshall(self) -> dict:
    return {
        "fqdn": self.FQDN,
        "variable": 49,
        "subcomponent": self.subcomponent.marshall(),
    }
```


### unmarshall
Once a marshalled object has been recieved, the unmarshall method may be used to
reverse the process. Like marshalling, unmarshalling should also be performed on
all subcomponents to fully resolve the object.

```python
@classmethod
def unmarshall(cls, registry: Registry, payload: dict) -> cls:
    # Note that registry.unmarshall will call cls.unmarshall on the class that
    # resolves to the subcomponent's fqdn.

    return cls(
        variable=payload["variable"],
        subcomponent=registry.unmarshall(payload["subcomponent"]),
    )
```

## Applicable
The ```Applicable``` interface provides the nessasary methods for objects to
perform manipulations against state.

### verify
The ```verify``` method should accept State and verify if the implementor is
capable of manipulating state either immediately or in the future. If
verification fails it should return a failed result with a reason why
verification failed.

```python
def verify(self, state: dict) -> Result:
    if self.from_address not in state["foo.bar"]:
        # self.from_address does not exist in 'foo.bar', in this example we
        # consider this a failure to verify and return a suitable message.
        return None, "from_addresss %s does not exist in 'foo.bar'" % self.from_address

    return None, None  # Nothing to return, but no error.
```

### apply
The ```apply``` method should manipulate and return state. In the event of an
error, return an errornous Result.

```python
def apply(self, state: dict) -> Result:
    try:
        state["foo.bar"][self.from_address] -= self.amount
        state["foo.bar"][self.to_address] += self.amount
        return state, None

    except Exception as e:
        return None, e.message
```


## Signable

The ```Signable``` interface provides a single method to take an unsigned object
and return a signed version.

### sign
The ```sign``` method should accept a ```credits.key.SigningKey``` and sign some
sort of challenge stored within the implementor. Then return a new instance of
the implementor with both the signature and the associated
```credits.key.VerifyingKey```. These additional variables can then for example
be used in conjunction with the ```Applicable.verify``` method to check if the
implementor has been signed.

```python
def sign(self, signing_key: SigningKey) -> self:
    verifying_key = signing_key.get_verifying_key()
    signature = signing_key.sign(self.challenge)

    return self.__class__(
        address=self.address,
        nonce=self.nonce,
        challenge=self.challenge,
        verifying_key=verifying_key,
        signature=signature,
    )
```


## Hashable

The ```Hashable``` interface provides a single method to provide a cryptographic
hash of the implementor.

### hash
Given a ```credits.hash.HashProvider```, construct some sort of string hash it.

```
def hash(self, hash_provider: credits.hash.HashProvider) -> str:
    # A primative example where we use variables from this class to construct a
    # challenge. Then hash and return the output.

    challenge = self.name + str(self.age) + str(self.value)
    return hash_provider.hash(challenge)
```
