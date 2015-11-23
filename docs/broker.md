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
will forget about its existence. Brokers are known from routers, like routers are
known from gateways. 

On the other hand, brokers communicate with a bunch of handlers that have registered themselves
beforehand. This way, when a device joins the network every broker has to communicate with its
own handlers list to determine wether or not it has to handle packets incoming from that
device. The previous assertion assumes that a given handler isn't registered to several
 brokers. 

Relations are schematically represented in the next diagram. 

-------------------
![Broker's relations](img/broker_relations.svg)
<p align="center">*Broker's relations*</p>
-------------------

For each node it is in charge of, the broker holds a network session key associated to the node.
It is thereby able to check the integrity of the packet by doing a `MIC check`. The process is
detailed in the [LoRaWAN specifications][lorawan] - section 4.4 Message Integrity Code (MIC). 

Depending of the packet's nature, the broker might communicate with either its network server
or a registered handler. Part of the packet's message contains a *MAC header* `MHDR` which is
not encrypted and gives details about the packet's nature (cf the [payload
cheatsheet](/img/cheatsheet.svg)). Thus, for any command (`FPort` set to `0`) the broker would
forward the action to its network server. Otherwise, the packet is forwarded to the right
handler. 

### Uplink transmissions

A broker receives transmissions from routers. Transmissions have two aspects:

- Discovering message
- Direct message

Discovering messages are sent when a router is trying to identify which brokers are in charge
of a node. The structure and the content of both discovering and direct messages are identical,
only the intent is different.  The direct message is addressed to a broker which is known
responsible for the packet. In both cases nevertheless, the broker will handle the packet in
same way, but the answer given to the router will differ. 

The broker has to perform a `MIC check` on the packet with a double goal:

- To ensure the packet is correct and valid
- To prevent from collided packet sent by mistake

Thus, because the node address is likely to collide with another node address, the broker might
receive packets that are not under its control. Using the network session key known for the
related node (one network session key per node), it can check wether or not it should handle
the packet. Having a concordance with both the `MIC` and the `DevAddr` (double collision) has
to be almost impossible and we'll consider that this is verified in practice (//TODO shall we
compute chances ?).

### Packet with commands

For any uplink messages, the broker will also have to notify its network server as an uplink
messages means that receive windows will be available for the device. A network server could
use one of those window to send a specific command to the node. 

Incidentally, when the received message is carrying a command, the command has to be
communicated to the network server as well. In most cases, no handler is reached and the
packet is completely handled by the network itself; the communication is thereby invisible for
the application. For some commands nevertheless (a `join request` for instance), 

### Packet with data

Also, in some cases, the application might want to use these receive windows to send back some
data to the node. Data from the application and commands can be merged in a single packet.
Waiting for the response of both entrants and doing a merge is under the broker
responsibility. 



[lorawan]: https://www.lora-alliance.org/portals/0/specs/LoRaWAN%20Specification%201R0.pdf
