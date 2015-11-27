Router
======

R          | Description
-----------|------------
R-1        | Forward uplink messages to brokers
R-2        | Forward downlink messages to gateways

R-1        | Description
-----------|------------
R-1.1      | Listen and receive incoming uplink transmissions
R-1.2      | Decode an incoming transmission 
R-1.3      | Lookup a device address
R-1.4      | Forward a packet to broker(s)

R-1.3      | Description
-----------|------------
R-1.3.1    | Lookup local storage addresses
R-1.3.2    | Invalidate expired entries

R-1.4      | Description
-----------|------------
R-1.4.1    | Send a packet to known brokers
R-1.4.2    | Broadcast a packet to all brokers
R-1.4.3    | Store device address and associated broker(s)

R-2        | Description
-----------|------------
R-2.1      | Listen and receive incoming downlink transmissions
R-2.2      | Determine recipient gateway(s)
R-2.3      | Forward a packet to gateway(s)

## Role

The router's role is straightforward; It has to transfer packets coming from an emitter to the
right recipient. The emitter and the recipient could be either:

- A gateway and a broker
- A broker and a gateway

This implies the router is able to interact with gateways and brokers in a bi-directional
way. Let's distinguish communications the following way:

- Uplink communication (from a gateway to one or several brokers)
- Downlink communication (from a broker to a gateway)

Communications protocols used between, on the one hand, gateways and the router (uplink) and,
on the other hand, the router and brokers (downlink) don't have to be identical. 

Both communication processes have their own characteristics and behavior, they will be detailed
separately. 

### Uplink communication

A given gateway will be connected to a router of its choice, meaning that the gateway is
configured to interact with that precise router (a gateway could be configured for several
routers, but the idea is the same; they are not allowed dynamically and won't change as long as
the configuration remains the same). Obviously, a router might received communications from
several gateways as well. Thus, gateways are completely unknown from a router - and would
remain unknown during the router lifecycle.

A router is thereby a machine on which gateway will attempt to connect. This assumes that the
router is accessible via a static IP address or solvable through a DNS service. The whole
protocol used by gateways can be found [here][gateway_protocol] and could be sum up as follow:

- Gateways initiate communication with a router
- Gateways send data using a json structure and containing one or several packets
- The router acknowledge reception of data
- Gateways could be protected by a firewall or could use a NAT, routers cannot initiate communications
- Gateways might trigger and pull the router periodically to keep a connection open (this is
  fairly an implementation detail, but the gateway protocol we are refering to is describing an
  implementation. We'll see how we handle this in the next section)
- The communication is closed after a delay (after the second receive window, cf. downlink
  communication)

Once a communication is established with a gateway and a packet received, the router has to
determine to which broker should the packet be forwarded. Two options exist:

- The address is known and associated to a broker
- The address is unknown

The former option will lead to a direct forwarding whereas the latter will require a network
discovering / broadcasting (see the section *Address resolution and caching*).

### Downlink communication

Because of the first version, the network will only supports devices of class A, the connection
between a gateway and a router would stay opened for a maximum of 2 seconds. The whole process
is detailed in the [LoRaWAN specifications 1.0][lorawan] - section 3.3 Receive Windows. In few
words though, after having emitted messages, a class A node will open two short receive windows
and will allow incoming messages from a gateway. Windows are opened exactly one and two seconds
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

### Address resolution and caching

A router has a configured list of brokers addresses as well as a local cache associating device
adresses to brokers. Entries of the cache get invalidated at regular interval (to determine
during testing or by computing), meaning that they are removed from the cache.  
When receiving a message, the router is in charge of looking into its local cache to determine
wether or not a broker is known for the related node and segment. This is done by broadcasting a
message to each known broker and by listening to their answers for the given message. 

The router then stores each broker able to handle messages coming from the given node. However,
because:

- Handlers may register after a node emits a signal to brokers
- A broker might not be available at the moment the broadcasting is done
- Two node addresses are in collision

The cache has to be invalidated to allow the network to recover a consistent state. Such that
no mapping between an address and a broker is settled forever after a first broadcast. However,
when an address is known and is not invalidated, any issue listed above will remain until the
next invalidation. 

## Interfaces 

In order to communicate with the outside world, we'll split the router in three parts: 

- The core router
- An uplink adapter
- A downlink adapter

The idea is to avoid to tightly couple the router core features with the way it is
communicating. By doing such a separation of concerns, we allow the router to evolve and change
its communication protocol at any moment, without any impact on the core mechanisms. Thus, as
long as the adapters remain compliant to a given interface they can be switched on demand. 

### Uplink adapter

We consider the following methods for the Uplink adapter (hereby known as `UpAdapter`):

```haskell
-- Notify the gateway that the given packet has been received
ack :: UpAdapter, Packet -> Unit

-- Send a downlink packet to a gateway
forward :: UpAdapter, Packet -> Unit
```

The uplink adapter is thereby in charge of queuing incoming packets, and trigger the router to
handle them properly. Because data coming from a gateway aren't formatted in the right way and
also because a gateway might send several packets through the same message, it is under the
uplink adapter responsability to decode and interpret the data accordingly. 

### Downlink adapter

We consider the following methods for the Downlink adapter (hereby known as `DownAdapter`):

```haskell
-- Send a packet to every available brokers. This method should trigger 
-- back some calls on the router to inform the router about which Broker 
-- are indeed responsible for the Packet.
broadcast :: DownAdapter, Packet -> Unit

-- Forward an uplink packet to a list of brokers using their addresses
forward :: DownAdapter, BrokerAddr[], Packet -> Unit
```

The downlink adapter should implement mechanism to handle network discovering (`broadcast`).
Basically, this will be done when there is no known broker for a given packet. The downlink
adapter is thereby in charge of registering device addresses to the core router once brokers
have been discovered. 

### Core

The core router (hereby known as `Router`) handle all the router logic. It also supplies a
concise interface to allow both adapters to trigger actions. Any error or success from both
adapters could trigger one of the following method.

``` haskell
-- Ask the router to handle a specific error. 
handleError :: Router, Error -> Unit

-- Handle an incoming uplink packet
handleUplink :: Router, Packet -> Unit

-- Handle an incoming downlink packet
handleDownlink :: Router, Packet -> Unit

-- Register a bunch of brokers address for a given device
registerDevice :: Router, DeviceAddr, BrokerAddr[] -> Unit
```

The `handleError` method gives adapters a way to notify the router of an unresolved transaction
or an incorrect behavior from the network. Errors are detailed below and should be explicit
enough to allow the router to recover from it. 

## Flow Chart

//TODO

## Errors

Errors types the router may encounter are listed right below. This list isn't settled and is
likely to grow during the development. However, it gives an overview of referenced errors. By
convention, all error names are written in [*snake_case*][snakecase].

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


