Common
======


## Packet representation
As long as all components use compatible representation of a packet, they sould be able to
communicate. Therefore, when refering to a `Packet`, we're seemingly refering a `json-like`
structure of the following shape:

```json
{
    "time": <String>,
    "tmst": <Number>,
    "freq": <Number>,
    "chan": <Number>,
    "rfch": <Number>,
    "stat": <Number>,
    "modu": <String>,
    "datr": <Number>, // In case of GFSK modulation
    "datr": <String>, // In case of LoRa modulation
    "codr": <String>,
    "rssi": <Number>,
    "lsnr": <Number>,
    "size": <Number>,
    "data": <String>
}
```

More information about the meaning of those fields could be found in the [semtech protocol
description][gateway_protocol].

## MAC Command
//TODO

## Errors

Any error coming from any components should provide the following information:

- a name / identifier 
- a date (the moment it happens)
- a message / description
- the packet or data manipulated if any

A given error will thus provide methods that reflect those attributes:

```haskell
-- Retrieve the error's name
name :: Error -> String

-- Retrieve the error's creation date
date :: Error -> Date|String

-- Retrieve the error's message
message :: Error -> String

-- Retrieve the error's data
params :: Error -> a
```


[gateway_protocol]: https://github.com/TheThingsNetwork/packet_forwarder/blob/master/PROTOCOL.TXT
