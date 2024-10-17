OSPFv2 is used on IPv4 
OSPFv3 is used on IPv6

* Link state routing protocol that was developed as an alternative for Routing Information * Protocol (RIP)
* Several benefits over RIP such as faster convergence and scales better
* Link state routing protocol using the concept of *areas*
* A Link is a interface on a router
* A Link can also be a network segment that connects two routers or a stub network such as Ethernet LAN
# OSPFv2
## Components
### Routing Protocol Messages
* Five packet types:
#### Hello Packet
* Discovers neighbours and builds adjacencies between them
* Advertise parameters on which two routers must agree on to become neighbours
* Elect the designated router (DR) and Backup Designated Router (BDR) on multiaccess networks like Ethernet. Point-to-point links do not require DR or BDR
![[Pasted image 20241017201755.png]]
#### Database description packet
* Contains an abbreviated list of the LSDB of the sending router and is used by the receiving routers to check against the local LSDB. The LSDB must be identical on all link-state routers within an area to construct an accurate SPF tree.
####  Link-state request (LSR) packet
* Receiving routers can request more information about any entry in the DBD by sending an LSR.
#### Link-state update packet
* This is used to reply to LSRs and to announce new information.
* LSUs contain several different type of LSAs.

| LSA Type | Description                                              |
| -------- | -------------------------------------------------------- |
| 1        | Router LSAs                                              |
| 2        | Network LSAs                                             |
| 3 or 4   | Summary LSAs                                             |
| 5        | Autonomous System External LSAs                          |
| 6        | Multicast OSPF LSAs                                      |
| 7        | Define for Not-So-Stubby Areas                           |
| 8        | External Attributes LSA for Border Gateway Patrol (BGPs) |
#### Link-state acknowledgment packet
* When an LSU is received the router sends an LSAck to confirm the receipt of the LSU. 
### Data Structures
* OSPF messages are used to create 3 databases:
1.  **Adjacency database** - This creates the neighbour table.
2. **Link-state database (LSDB)** - This creates the topology table.
3. **Forwarding database** - This creates the routing table.

| Database            | Table           | Description                                                                                                                                                                         |
| ------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Adjacency Database  | neighbour Table | \* List of all neighbour routers to which a router has a established bidirectional communication<br>\* Unique for each router<br>* `show ip ospf neighbor`                          |
| Link State Database | Topology table  | \* Lists information about all other routers in the network.<br>\* Represents the network Topology<br>\* All routers in an area have an identical LSDB<br>\*`show ip ospf database` |
| Forwarding Database | Routing Table   | * List of routes generate when an algorithm is run on a LSDB <br>\* Routing table of each router is unique and contains information on how and where to send packets                |

* The router builds the topology table using results of calculations using Dijkstra shortest-path first (SPF) algorithm
	* Creates an SPF tree by placing each router at the root for the tree and calculating the shortest path to each node 
	* The best routes get put into the forwarding database
### Link-State Operation
* OSPF routers complete a generic link-stating routing process to reach a state of convergence.
1. Establish Neighbour Adjacencies 
2. Exchange Link-state Advertisements
3. Build the link state database
4. Execute the SPF algorithm
5. Choose best route
#### Establish Neighbour Adjacencies 
* OSPF enabled routes need to recognise each other on the network before they can share information. 
1. OSPF-enabled router sends Hello packets to all OSPF-enabled interfaces,
	* Determines if neighbours are present on this link.
2. if neighbours are present the OSPF-enabled router attempts to establish a link.  
#### Exchange Link-state Advertisements
1. After the hellos routers send link-state advertisements (LSAs)
2. These contain the cost of each directly connected link.
3. Routers flood their LSAs to adjacent neighbours
4. Adjacent neighbours receiving LSA immedicably flood the LSA to other directly connected routers until all routers have all the LSAs.
#### Build the link state database
1. After all LSA's are received, the routers build the topology table (LSDB)
2. The database holds all information about the topology 
#### Execute the SPF Algorithm
1. Routers then execute the SPF algorithm, this creates the SPF tree. 
#### Choose the best Route.
1. After the SPF tree is build, the best paths to each network are offered to the IP routing table.
2. The route will be added to the routing table unless their is a static source to the same network.
![[Pasted image 20241016203805.png]]
### Single-Area and Multiarea
* OSPF supports hierarchical routing using areas.
	* Makes it more efficient and scalable 
#### Single-Area OSPF
* All routes are in one area, best practise is use area 0
#### Multiarea OSPF
* OSPF is implemented using multiple areas, in a hierarchal fashion, all areas must be connected to the backbone area (area 0). Routes interconnection the areas are referred to as Area border routers.
![[Pasted image 20241016204139.png]]

* One large routing domain can be divided into smaller areas. 
* Allow for routing to occur across areas but database calculation and other computationally heavy stuff to be kept within area.
##### Benefits: 
* *Smaller routing tables* - Tables are smaller because of fewer routes, network addresses can be summarised between areas.
* *Reduced link-state update overhead* - Designing multiarea OSPF with smaller areas minimised processing and memory requirements
* *Reduced frequency of SPF calculations* - Localises the impact of topology changes. 
# OSPFv3
* Equivalent for IPv6.
* Exchanges routing information to populate the IPv6 routing table with remote prefixes. 
* Support for both IPv4 and v6 using OSPF address family feature.
* Uses IPv6 as the network layer transport. 
* Uses the SPF algorithm
![[Pasted image 20241016204947.png]]
