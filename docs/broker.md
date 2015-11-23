Broker
======

## Role

Brokers constitute the heart of the network. They hold the logic that makes every other
component work together (routers, network servers and handlers). They play a role of mediator,
making sure that packets are correct and handling communications between other components. To
each broker is associated a given network server with which it is working closely. We can see
the network server as external ressources of computing and storage to which a broker could
refer at any time. 

Therefore, a broker is in charge of a set of nodes. This is a 1-to-1 relation meaning that a
node is controlled by a unique broker. There is for the moment no duplication or sharing
possible. A broker does not have access to network servers different from the one it has been
assigned. A node isn't shared and all packets coming from a given node end to the same
given broker.

Besides, a broker does not know any router by advance. The process is seemingly similar to the way
gateways and routers are getting to know each others. A router will initiate a communication
toward a broker. After dealing with the communication, the broker may reply to the router and
will forget about its existence after that. Brokers are known from routers, like routers are
known from gateways. 

//TODO handlers

For each node it is in charge of, the broker hold a network session key associated to the node.
It is thereby able to check the integrity of the packet by doing a `MIC check`. The process is
detailed in the [LoRaWAN specifications][lorawan] - section 4.4 Message Integrity Code (MIC). 

Depending of the packet's nature, the broker might communicate with either its network server
or a registered handler.

### Uplink communications

Also, a broker receive transmissions from routers. Transmissions have two aspects:

- Discovering message
- Direct message

### Downlink communication

### Network Commands

[lorawan]: https://www.lora-alliance.org/portals/0/specs/LoRaWAN%20Specification%201R0.pdf
