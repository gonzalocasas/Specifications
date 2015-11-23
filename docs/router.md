Router
======

## Role

The router's role is straightforward; It has to transfer packet coming from an emitter to the
right recipient. The emitter and the recipient could be either:

- A gateway and a broker
- A broker and a gateway

This implies the router is able to interact with gateways and brokers in a bi-directional
way. Let's distinguish communications the following way:

- Uplink communication (from a gateway to one or several brokers)
- Downlink communication (from a broker to a gateway)

Communications protocols used between, on the one hand, gateways and the router (uplink) and,
on the other hand, the router and brokers (downlink) don't have to be identical. 

Both communication process have their own characteristics and behavior, they will be detailed
separately. 

### Uplink communication

A given gateway will be connected to a router of its choice, meaning that the gateway is
configured to interact with that precise router. A router nevertheless might received
communication from several gateways. Thus, gateways are completely unknown from a router - and
would remain unknown during the router lifecycle.

A router is thereby a machine on which gateway will attempt to connect. This assumes that the
router is accessible via a static IP address or solvable through a DNS service. The whole
protocol used by gateways can be found [here][gateway_protocol] and could be sum up the
following ways:

- Gateways initiate communication with a router
- Gateways send data using a json structure and containing one or several packets
- The router acknowledge reception of data
- Gateways could be protected by a firewall or could use a NAT, routers cannot initiate communications
- Gateways might trigger and pull the router periodically to keep a connexion open

### Downlink communication

As for the first version, the network will only supports devices of class A, the connexion
between a gateway and a router would stay opened for a maximum of 2 seconds. The whole process
is detailed in the [LoRaWAN specifications 1.0][lorawan] - section 3.3 Receive Windows. In few
words though, after having emitted messages, a class A node will open two short receive windows
and allow incoming messages from a gateway. Windows are opened exactly one and two seconds
after the packet's emission. Sending packet at the right time is part of the gateway
responsibility, however, scheduling the emission is part of the network's one. 

If any command or data have to be sent to the node, it has to be done precisely during one of
this two windows. More details about commands are given in the section related to the
Network Server. Incidentally, only one of the two windows is available, meaning that sending
through the first window will cause the second one to never be opened. 

-------------------
![Receive Windows](img/receive_windows.svg)
<p align="center">*Receive Windows*</p>
-------------------


Having said that, a downlink communication could be initiated by either an Application or a
Network Server in response of an uplink message from a device. The role of a router is to
forward those messages, not to trigger or schedule them.

## Decoupling

In order to communicate with the outside world, we'll split the router in three parts: 

- The core router
- An uplink adapter
- A downlink adapter

The idea is to avoid to tighly couple the router core features with the way it is
communicating. By doing such a separation of concerns, we allow the router to evolve and change
its communication protocol at any moment, without any impact on the core mechanisms. Thus, as
long as the adapters remain compliant to a given interface they can be switched on demand. 

All three components will share a common representation of a packet tagged with a version
number. As long as all component use compatible representation of a packet, they sould be able
to communicate. 

Packets are `json` structure with the following structure:

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
    "lsnr": <Number>,
    "size": <Number>,
    "data": <String>
}
```

More information about the meaning of those fields could be found in the [semtech protocol
description][gateway_protocol].


## Uplink adapter

We consider the following methods for the Uplink adapter (hereby known as `UpAdapter`):

```haskell
-- Notify the gateway that the given packet has been received
ack :: UpAdapter, Packet -> Unit
ack (adapter, packet)

-- Send a downlink packet to a gateway
forward :: UpAdapter, Packet -> Unit
forward (adapter, packet)
```

The uplink adapter is thereby in charge of queuing incoming packet, and trigger the router to
handle them properly. Because data coming from a gateway aren' formatted in the right way and
also because a gateway might send several packets through the same message, it is under the
uplink adapter responsability to decode and interpret the data accordingly. 

## Downlink adapter

We consider the following methods for the Downlink adapter (hereby known as `DownAdapter`):

```haskell
-- Send a packet to every available brokers. This method should trigger 
-- back some calls on the router to inform the router about which Broker 
-- are indeed responsible for the Packet.
broadcast :: DownAdapter, Packet -> Unit
broadcast (adapter, packet)

-- Forward an uplink packet to a list of brokers using their addresses
forward :: DownAdapter, BrokerAddr[], Packet -> Unit
forward (adapter, [broAddr1, broAddr2], packet)
```

The downlink adapter should implement mechanism to handle network discovering (`broadcast`).
Basically, this will be done when there is no known broker for a given packet. The downlink
adapter is thereby in charge of registering device addresses to the core router once brokers
have been discovered. 

## Core

The core router (hereby known as `Router`) handle all the router logic. It also supplies a
concise interface to allow both adapters to trigger actions. Any error or success from both
adapters might trigger one of the following method.

``` haskell
-- Ask the router to handle a specific error. 
handleError :: Router, Error -> Unit
handleError (router, err)

-- Handle an incoming uplink packet
handleUplink :: Router, Packet -> Unit
handleUplink (router, packet)

-- Handle an incoming downlink packet
handleDownlink :: Router, Packet -> Unit
handleDownlink (router, packet)

-- Register a bunch of brokers address for a given device
registerDevice :: Router, DeviceAddr, BrokerAddr[] -> Unit
registerDevice (router, devAddr, [broAddr1, broAddr2])
```

The `handleError` method gives adapters a way to notify the router of an unresolved transaction
or an incorrect behavior from the network. Errors are detailed below and should be explicit
enough to allow the router to recover from it. 

## Errors

Any error coming from any router's components should supply the following information:

- a name / identifier 
- a date (the moment it happens)
- a message / description
- the packet or data manipulated if any

A given error will thus provide methods that reflect those attributes:

```haskell
-- Retrieve the error's name
name :: Error -> String
name (err)

-- Retrieve the error's creation date
date :: Error -> Date|String
date (err) 

-- Retrieve the error's message
message :: Error -> String
message (err)

-- Retrieve the error's data
params :: Error -> a
params (err)
```

Errors types the router may encounter are listed right below. This list isn't settled and is
likely to evolve and change during the development. However, it gives an overview of referenced
errors. By convention, all error names are written in [*snake_case*][snakecase].

name                   | description
-----------------------|-------------
invalid\_packet        | The given packet has a bad format
connection\_lost       | The connection with a recipient has been lost 
no\_response           | No response received from a recipient
unable\_forward\_up    | Unable to forward a packet (uplink)
unable\_forward\_down  | Unable to forward a packet (downlink)

[gateway_protocol]: https://github.com/TheThingsNetwork/packet_forwarder/blob/master/PROTOCOL.TXT
[lorawan]: https://www.lora-alliance.org/portals/0/specs/LoRaWAN%20Specification%201R0.pdf
[snakecase]: https://en.wikipedia.org/wiki/Snake_case


