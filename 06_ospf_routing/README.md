# Open Shortest Path First (OSPF) Routing

## Basic OSPF Overview

### Motivation for OSPF

__Limitations of RIP (Distance Vector)__

- __Slow convergence__ - prone to routing loops
- __Max 15 hops__ - unsuitable for large networks
- __Flat design__ - no suport for hierarchy
- __One metric only__ - hop count

__OSPF: An Open Link-State Solution__

- __Fast convergence__  via Dijkstra's algorithm
- __Hierarquical ares__ reduce fooding and complexitry
- __Support multiple metrics__ - cost based on bandwidth
- __Built-in route summarization and authentication__

#### Main Advantages of OSPF

- No hop limit
- Suports classless routing
- Reduced signalling traffic withg long update intervals or topology-triggered changes
- Fast convergence
- Loop-free routing
- Reacts quickly to network changes
- Better load balancing
- Logica area segmentation: "divide to conquer"
- External route tagging
- Authentication support

### Scalability of Autonomous Systems and Hierarchical Areas

Knowing all routes implies:

- High memory capacity
- Information exchange about all routes
- Routers exchange info with all others

Computational complexity grows linearly with network size

__Solution__:

- Divide the network into hierarchies
- __Use different routing algorithms for different context__

### OSPF Operation Stages

![OSPF Operation Stages](./img/01.png)

### OSPF and Link-State Routing

Each __OSPF router__ builds and mantains a map of area'a topology:

- Exchange __Link State Updates (LSUs)__ to advertise directly connected links
- Uses __multicast__ (224.0.0.5/6) for efficient communication
- Stores the learned topology in the __Link-State Database (LSDB)__

Routers use __Dijkstra's SPF algorithm__ to compute the shortest path tree:

- SPF is recalculated whenever the LSDB changes
- Results populate the __routing table__

__Hello messages__ are exchanged periodically to maintain neighbor relationships:

- Loss of Hello triggers flooding and SPF recalculation

The routing tables reflects:

- __Intra-area__ SPF paths
- __Inter-area__ routes from ABRs
- __External__ routes from ASBRs

### OSPF Network Types - Behavior and Motivation

__Why define different network tyoes in OSPF?__

- OSPF adapts its behavior based on characteristics of the underlying Layer 2 technology
- Different network types affect how neighbors are discovered, how LSAs are exchanged, and whether Designated Routers (DR/BDR) are needed
- Proper configuration ensures convergence efficiency and avoids unnecessary adjacencies

| Interface Type | Uses DR/BDR | Hello Int. | Needs _neighbor_ cmd? | >2 Hosts Supported |
| -------------------------------------| --- | -- | --- | --- |
| __Broadcast (BMA) - e.g., ethernet__ | Yes | 10 | No  | Yes |
| __Non-Broadcast (NBMA)__             | Yes | 30 | Yes | Yes |
| __Point-to-Multipoint (PTM)__        | No  | 30 | No  | Yes |
| __PTM Non-Broadcast__                | No  | 30 | Yes | Yes |
| __Point-to-Point (Serial)__          | No  | 10 | No  | No  |
| __Loopback__                         | No  | -  | -   | No  |

### OSPF Packet Types - Roles in the Protocol

__Why does OSPF use multiple packet types?__

- OSPF is a reliable link-state protocol that requires coordinated neighbor discovery, database exchange, and update acknowledgment.
- Each packet type has a specific role in building, synchronizing, and maintaining the LSDB (Link-State Database)
- Separating functions improves efficiency, control, and convergence accuracy

| Type | Packet Name | Description |
| ---- | ----------- | ----------- |
| 1    | __Hello__   | Discover neighbors and establish adjacencies |
| 2    | __DBD__     | Database Description — check for LSDB synchronization |
| 3    | __LSR__     | Link-State Request — request specific LSAs |
| 4    | __LSU__     | Link-State Update — send LSAs to neighbors |
| 5    | __LSack__   | Acknowledge receipt of Link-State Updates |

### OSPF Neighbors and Adjacencies

__Neighbor__

- Two routers are __neighbors__ if they have interfaces on the same network (Layer 3 reachable)
- Neighbor relationships are formed and maintained via the __Hello protocol__
- Routers only become neighbors if these match:
	+ __Area ID (area/id)__
	+ __Authentication parameters__
	+ __Hello and Dead intervals__
	+ __Stub area flag__
	
__Adjacency__

- An adjacency is a __deeper relationship__ between two neighbors to exchange and synchronize LSDBs
- Not all neighbors become adjacent
- Routers that are adjacent maintain identical link-state databases (if in the same area)

### Router ID in OSPF

__Definition__ - the __Router ID (RID)__ is a 32-bit number assigned to uniquely identify each OSPF router within an Autonomous System

__Selection Rules__ - The OSPF standard recommends using the __lowest IP address__ among the router’s interfaces. However, __some implementations__ (e.g., Cisco) use the __highest IP address__ instead.

__RID Selection Priority__

1. Manually configured RID (if set)
2. Highest IP address among __loopback interfaces__
3. Highest IP address among __physical interfaces__

### Designated Router (DR) in OSPF

__Why is a DR needed in OSPF?__

- On broadcast (BMA) and NBMA networks, a __Designated Router (DR)__ is elected when more than 2 routers share the segment
- DR election reduces the number of required adjacencies and minimizes OSPF LSA traffic
- __Without a DR__: _n_ routers form _n(n − 1)/2_ adjacencies. 
- __With a DR__: each router only forms adjacency with the DR (and optionally BDR)

__Key Characteristics__

- DRs originate __Type 2 LSAs (network LSAs)__
- All routers on the segment become neighbors, but only DR and BDR form adjacencies with others
- Point-to-point links do __not__ use DR/BDR

__Benefit__: reduces OSPF-related traffic and improves scalability on multi-access networks

### DR/BDR Election Criteria

__Router Priority__:

- Configurable (0–255), default is 1
- Value 0 makes the router __ineligible__ for election
- Highest priority wins. In case of tie, highest Router ID is used

__Router ID (RID)__:

- Highest IP of loopback interfaces is chosen
- If no loopback, highest IP among active interfaces is used
- Can be manually configured

![DR/BDR Election Criteria](./img/02.png)

### Loopback Interfaces in OSPF

- __Stability__: Loopback interfaces are only administratively enabled/disabled, so the Router ID remains consistent
- __Preference__: If at least one loopback exists, its IP address is used for the Router ID rather than that of a physical interface
- __LSA Behavior__: Loopbacks are advertised via __Router-LSAs__ as single host routes, appearing as directly reachable destinations

![Loopback Interfaces](./img/03.png)

---

## OSPF Areas and Router Types

### OSPF Autonomous Systems and Areas

__Autonomous System (AS)__:

- A group of routers that share routing information using the same protocol — in this case, OSPF
- Typically managed by a single administrative entity

__Motivation for Dividing into Areas__:

- An AS can be divided into smaller groups called __areas__
- This segmentation hides the internal topology of each area from others
- Instability in one area does not affect the rest of the network
- Routers maintain fewer routes and require less memory
- Routes between areas can be summarized, reducing routing table size

> Each OSPF router belongs to exactly one area, but ABRs connect multiple areas

### OSPF Area Splitting Concept

- Network can be segregated using __Areas__
- Traffic between areas must traverse __Area 0__ (Backbone)
- Avoid the exchange of Link State Advertisements (__LSAs__) across too many subnets:
	+ Reduces CPU and memory usage
	+ Improves convergence time
- Each area maintains its own Link State Database (__LSDB__)
- Route summarization at Area Border Routers (__ABRs__) reduces domain-wide routing entries
- Autonomous Systems Border Routers (__ASBRs__) redistribute foreign routes into OSPF

![OSPF Areas](./img/04.png)

### Types of OSPF Routers

__Internal Router__

- Connected only to routers within the same area

__Area Border Router (ABR)__

- Connects routers from multiple areas (including the backbone)
- Responsible for exchanging routing information between areas
- Summarizes external area costs and injects them into its area
- After SPF is computed, inter-area routes are derived from ABR summaries

__Autonomous System Border Router (ASBR)__

- Connects to routers in other autonomous systems
- May run other routing protocols (IGP or EGP — e.g., RIP, EIGRP, BGP)

__Backbone Router__

- Has at least one interface running OSPF in area 0

### Inter-Area Communication and Backbone Role

- Areas advertise route summaries using a __distance-vector algorithm__
- To prevent routing loops, all inter-area communication passes through __Area 0 (backbone)__
- The backbone ensures a central point of coordination across areas
- When a direct connection to Area 0 is not possible, __virtual links__ allow remote areas to connect
	+ A virtual link is a logical tunnel between two ABRs, using a transit area to reach the backbone.
	+ The transit area must be fully adjacent (full OSPF adjacency) to allow the tunnel.
	
![Virtual Link](./img/05.png)

### Link State Database (LSDB)

- Each area maintains its own LSDB — a collection of:
	+ Router LSAs (used by the Dijkstra algorithm)
	+ Network LSAs (used by the Dijkstra algorithm)
	+ Summary LSAs
	+ AS External LSAs
- LSAs are only flooded within the area where they originate
- AS External LSAs are also part of the local __LSDB__, even if they describe external routes
- From the LSAs, each router builds a “map” of its area using the SPF algorithm
- __Important distinctions__:
	+ LSDBs are identical between routers in the same area
	+ LSDBs are different across areas due to summarization and distance-vector format of inter-area/external routes
	
![LSDB Components](./img/06.png)


### Intra-area Routing with SPF

- Each router uses its link-state database (LSDB) to build a __shortest-path tree__ rooted at itself
- This tree identifies the optimal path to every destination within the area
- All routers independently run the same SPF algorithm in parallel
- For correctness, all routers must maintain identical LSDBs
- External routes (inter-area or inter-AS) appear as leaves in the tree
	+ These are still link-state entries, but learned via external summarization
	
![Shortest Path Tree](./img/07.png)

### Differentiated Routing and Type of Service (TOS) in OSPF

- OSPF supports multiple metrics
	+ Default metric: inverse of link bandwidth
	+ Cost = ```(10^8) / link bandwidth (bps)```
- With a single metric, TOS (Type of Service) is not supported
- With multiple metrics, TOS-based routing becomes available:
	+ Separate routing tables for each combination of the 3 TOS bits:
		* _delay_
		* _throughput_
		* _reliability_
- __Example__:
	+ If TOS bits specify low delay, low throughput, and high reliability, OSPF computes a path accordingly
	
![Tipes of Service - IPv4 Header](./img/08.png)

### OSPF Configuration

OSPF can be configured in two alternative approaches:

1. __Using the network statement (global mode)__
	- Most common method
	- Matches interface3s using IP + wildcard mask
	- Configures under the OSPF process
	```txt
	Router(config)# router ospf 1
	Router(config-router)# network 10.1.2.2 0.0.0.0 area 0
	``` 
2. __Interface-specific configuration__
	- Applied directly to interfaces
	- Takes precedence in case of conflict
	```txt
	Router(config)# interface Serial1/1
	Router(config-if)# ip ospf 1 area 1
	```

![Interfaces - different areas](./img/09.png)

#### Configuration Using Wildcards

__What is a wildcard?__

- Used in `network` statements to match interface IPs
- `0.0.0.0` = exact match (most specific)
- `0.255.255.255` = broader match (more general)

__Examples__:

- `network 10.0.0.0 0.255.255.255` - enables all 10.x.x.x interfaces
- `network 10.2.3.2 0.0.0.0` - enables only exact interface

__OSPF activation vs advertisement__:

- OSPF activates __per interface__
- But LSAs advertise the __entire network segment__

> __OSPF is enabled per interface, but LSAs advertise the entire subnet!__

__Configuration Example__:

```txt
# Global OSPF configuration using network statement
Router ( config ) # router ospf 1
Router ( config - router ) # network 10.1.2.2 0.0.0.0 area 0.0.0.0

# Interface-specific configuration (overrides network cmd if both exist)
Router ( config ) # interface Serial1 /1
Router ( config - if ) # ip ospf 1 area 0.0.0.1

# Passive interface configuration to suppress OSPF Hellos
Router ( config - router ) # passive - interface Serial1 /0

# Check interface OSPF state and process bindings
Router # show ip ospf interface brief
Router # show ip protocols

# Show OSPF processes running on the router
Router # show ip protocols

# Display interfaces running OSPF with area info and metrics
Router # show ip ospf interface brief

# Detailed view for a specific interface
Router # show ip ospf interface GigabitEthernet0 /1

# View OSPF neighbors and their adjacency states
Router # show ip ospf neighbor

# View the LSDB (Link-State Database)
Router # show ip ospf database

# Display the IP routing table filtered by OSPF entries
Router # show ip route ospf

# Restart the OSPF process (resets all neighbors)
Router # clear ip ospf process

# Confirm before resetting the OSPF process
Reset ALL OSPF processes ? [ no ]: yes
```
