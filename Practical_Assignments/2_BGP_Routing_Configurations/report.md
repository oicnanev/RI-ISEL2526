# Lab Project N.º 2 - BGP ROUTING CONFIGURATIONS

> REDES DE INTERNET (RI) 2025-2026

---

## Group 1 - Class 51D

### Professor Luís Mata

> luis.mata@isel.pt

- 45824 Nuno Venâncio
- 49420 André Carrilho
- 51454 Hugo Leal

![Lab Project 2 topology](./assets/img/01.png)

<div style="page-break-after: always"></div>

## Index

- Introduction
- 1 - Phase 1 - IGP Configuration
	+ 1.1 - Objectives
	+ 1.2 - Implementation
	+ 1.3 - Test and Validation
	+ 1.4 - Practical Questions
- 2 - Phase 2 - iBGP and eBGP Without Routing Policies
	+ 2.1 - Objectives
	+ 2.2 - Implementation
	+ 2.3 - Test and Validation
	+ 2.4 - Practical Questions
- 3 - Phase 3 - Routing Policies Implementation
	+ 3.1 - Objectives
	+ 3.2 - Implementation
	+ 3.3 - Test and Validation
	+ 3.4 - Practical Questions
- 4 - Phase 4 - Influence the Internet Routing
	+ 4.1 - Objectives
	+ 4.2 - Implementation
	+ 4.3 - Test and Validation
	+ 4.4 - Practical Questions
- 5 - Phase 5 - Security Practices
	+ 5.1 - Objectives
	+ 5.2 - Implementation
	+ 5.3 - Test and Validation
	+ 5.4 - Practical Questions
- Conclusion
- Appendices
	+ IP Addressing for Lab Topology
	+ TCLSH (Tool Control Language) Shell for Testing
	
<div style="page-break-after: always"></div>

## Introduction

__TODO__

<div style="page-break-after: always"></div>

## 1 - Phase 1 - IGP Configuration

### 1.1 Objectives

- Configure the IP addresses for all the interfaces
- Configure the OSPF as per the design rules on the ASes
- Implement the OSPF multi-area topology in AS 5511: area 0 and area 1 (stub)
- Test the private infrastructure connectivity’s inside the ASes
- Test the interfaces connectivity between the ASes

### 1.2 Implementation

We assign IP addresses to all interfaces according to table 1 in Appendice _IP Addressing for Lab Topology_

Router example:

```txt
interface Loopback0
 ip address 10.1.1.1 255.255.255.255
!
interface Loopback1
 ip address 48.73.239.11 255.255.255.255
!
interface GigabitEthernet1/0
 ip address 10.1.2.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet2/0
 ip address 10.1.3.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet3/0
 ip address 48.73.240.1 255.255.255.252
 no shutdown
```

Server example:

```txt
ip 130.41.46.1 130.41.46.2 30
set pcname Server1
```

#### OSPF Configuration

__AS 1273__ - Vodafone and __AS 17390__ - IBM

```txt
! All Routers -------------------------------------------------------------
router ospf 1
 passive-interface Loopback0               
 network 10.0.0.0 0.255.255.255 area 0        
 log-adjacency-changes
```

__AS 5511__ - FTRSI (Orange - Worldwide IP Backbone)

```txt
! ROUTER 10 ---------------------------------------------------------------
router ospf 1
 router-id 10.10.10.10             
 network 10.10.11.0 0.0.0.3 area 0   ! R11
 network 10.10.12.0 0.0.0.3 area 1   ! R12
 network 10.10.10.10 0.0.0.0 area 0  ! Lo0
 area 1 stub        
 log-adjacency-changes

! ROUTER 11 ---------------------------------------------------------------
router ospf 1
 router-id 10.11.11.11              
 network 10.10.11.0 0.0.0.3 area 0    ! R10
 network 10.11.12.0 0.0.0.3 area 1    ! R12
 network 10.11.11.11 0.0.0.0 area 0   ! Lo0
 area 1 stub       
 log-adjacency-changes

! ROUTER 12 ---------------------------------------------------------------
router ospf 1
 router-id 10.12.12.12             
 network 10.11.12.0 0.0.0.3 area 1      ! R11
 network 10.10.12.0 0.0.0.3 area 1      ! R10
 network 10.12.12.12 0.0.0.0 area 1     ! Lo0
 network 46.87.162.0 0.0.0.255 area 1   ! Server5
 area 1 stub      
 log-adjacency-changes
```

### 1.3 Test and Validation

#### Test the private infrastructure connectivity’s inside the ASes

__AS 1273__ - Vodafone

```txt
! From Router 1 - Pings to all other AS routers Loopback 0 --------------

R1#ping 10.2.2.2 ! Router 2 ---------------------------------------------

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/24/72 ms

R1#ping 10.3.3.3 ! Router 3 ---------------------------------------------

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/16/36 ms

R1#ping 10.4.4.4 ! Router 4 ---------------------------------------------

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.4.4.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/26/40 ms

R1#ping 10.5.5.5 ! Router 5 ---------------------------------------------

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.5.5.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 24/25/32 ms

R1#ping 10.6.6.6 ! Router 6 ---------------------------------------------

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.6.6.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 24/36/52 ms
```

OSPF configuration analysis

```txt
! ROUTER 1 --------------------------------------------------------------
R1#sh ip ospf database

            OSPF Router with ID (48.73.239.11) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    1998        0x8000000A 0x004493 3
48.73.239.22    48.73.239.22    459         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    1969        0x80000009 0x008FD6 4
48.73.239.44    48.73.239.44    1993        0x80000009 0x003310 4
48.73.239.55    48.73.239.55    2000        0x80000008 0x006E2A 2
48.73.239.66    48.73.239.66    1512        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    459         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    1998        0x80000005 0x00F7BE
10.2.4.2        48.73.239.44    1729        0x80000004 0x00C2BA
10.3.4.2        48.73.239.44    1993        0x80000007 0x004B23
10.3.5.2        48.73.239.55    2000        0x80000007 0x006CEA
10.4.6.2        48.73.239.66    1511        0x80000004 0x002215

R1#show ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/3] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
                    [110/3] via 10.1.2.2, 01:46:27, GigabitEthernet1/0
O       10.2.2.2/32 [110/2] via 10.1.2.2, 01:46:27, GigabitEthernet1/0
O       10.3.3.3/32 [110/2] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
O       10.6.6.6/32 [110/4] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
                    [110/4] via 10.1.2.2, 01:46:27, GigabitEthernet1/0
O       10.3.5.0/30 [110/2] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
O       10.2.4.0/30 [110/2] via 10.1.2.2, 01:46:27, GigabitEthernet1/0
O       10.3.4.0/30 [110/2] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
O       10.4.4.4/32 [110/3] via 10.1.3.2, 01:37:27, GigabitEthernet2/0
                    [110/3] via 10.1.2.2, 01:46:27, GigabitEthernet1/0
O       10.5.5.5/32 [110/3] via 10.1.3.2, 01:37:27, GigabitEthernet2/0

R1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.33      1   FULL/BDR        00:00:37    10.1.3.2        GigabitEthernet2/0
48.73.239.22      1   FULL/DR         00:00:35    10.1.2.2        GigabitEthernet1/0

R1#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.1.1.1/32        1     LOOP  0/0
Gi2/0        1     0               10.1.3.1/30        1     DR    1/1
Gi1/0        1     0               10.1.2.1/30        1     BDR   1/1
```

```txt
! ROUTER 2 --------------------------------------------------------------
R2#sh ip ospf database

            OSPF Router with ID (48.73.239.22) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    61          0x8000000B 0x004294 3
48.73.239.22    48.73.239.22    521         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    21          0x8000000A 0x008DD7 4
48.73.239.44    48.73.239.44    18          0x8000000A 0x003111 4
48.73.239.55    48.73.239.55    12          0x80000009 0x006C2B 2
48.73.239.66    48.73.239.66    1573        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    521         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    62          0x80000006 0x00F5BF
10.2.4.2        48.73.239.44    1791        0x80000004 0x00C2BA
10.3.4.2        48.73.239.44    18          0x80000008 0x004924
10.3.5.2        48.73.239.55    12          0x80000008 0x006AEB
10.4.6.2        48.73.239.66    1573        0x80000004 0x002215

R2#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/2] via 10.2.4.2, 02:06:08, GigabitEthernet2/0
O       10.1.3.0/30 [110/2] via 10.1.2.1, 01:43:58, GigabitEthernet1/0
O       10.3.3.3/32 [110/3] via 10.2.4.2, 01:40:50, GigabitEthernet2/0
                    [110/3] via 10.1.2.1, 01:40:50, GigabitEthernet1/0
O       10.1.1.1/32 [110/2] via 10.1.2.1, 02:06:18, GigabitEthernet1/0
O       10.6.6.6/32 [110/3] via 10.2.4.2, 02:06:08, GigabitEthernet2/0
O       10.3.5.0/30 [110/3] via 10.2.4.2, 01:40:50, GigabitEthernet2/0
                    [110/3] via 10.1.2.1, 01:40:50, GigabitEthernet1/0
O       10.3.4.0/30 [110/2] via 10.2.4.2, 01:40:50, GigabitEthernet2/0
O       10.4.4.4/32 [110/2] via 10.2.4.2, 02:06:08, GigabitEthernet2/0
O       10.5.5.5/32 [110/4] via 10.2.4.2, 01:40:50, GigabitEthernet2/0
                    [110/4] via 10.1.2.1, 01:40:50, GigabitEthernet1/0
                    
R2#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.44      1   FULL/DR         00:00:32    10.2.4.2        GigabitEthernet2/0
48.73.239.11      1   FULL/BDR        00:00:36    10.1.2.1        GigabitEthernet1/0

R2#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.2.2.2/32        1     LOOP  0/0
Gi2/0        1     0               10.2.4.1/30        1     BDR   1/1
Gi1/0        1     0               10.1.2.2/30        1     DR    1/1
```

```txt
! ROUTER 3 --------------------------------------------------------------
R3#sh ip ospf database

            OSPF Router with ID (48.73.239.33) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    163         0x8000000B 0x004294 3
48.73.239.22    48.73.239.22    625         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    121         0x8000000A 0x008DD7 4
48.73.239.44    48.73.239.44    120         0x8000000A 0x003111 4
48.73.239.55    48.73.239.55    112         0x80000009 0x006C2B 2
48.73.239.66    48.73.239.66    1675        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    625         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    163         0x80000006 0x00F5BF
10.2.4.2        48.73.239.44    1893        0x80000004 0x00C2BA
10.3.4.2        48.73.239.44    120         0x80000008 0x004924
10.3.5.2        48.73.239.55    112         0x80000008 0x006AEB
10.4.6.2        48.73.239.66    1675        0x80000004 0x002215

R3#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/2] via 10.3.4.2, 01:42:37, GigabitEthernet1/0
O       10.2.2.2/32 [110/3] via 10.3.4.2, 01:42:37, GigabitEthernet1/0
                    [110/3] via 10.1.3.1, 01:42:37, GigabitEthernet2/0
O       10.1.2.0/30 [110/2] via 10.1.3.1, 01:42:37, GigabitEthernet2/0
O       10.1.1.1/32 [110/2] via 10.1.3.1, 01:42:37, GigabitEthernet2/0
O       10.6.6.6/32 [110/3] via 10.3.4.2, 01:42:37, GigabitEthernet1/0
O       10.2.4.0/30 [110/2] via 10.3.4.2, 01:42:37, GigabitEthernet1/0
O       10.4.4.4/32 [110/2] via 10.3.4.2, 01:42:37, GigabitEthernet1/0
O       10.5.5.5/32 [110/2] via 10.3.5.2, 01:42:37, GigabitEthernet3/0

R3#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.55      1   FULL/DR         00:00:30    10.3.5.2        GigabitEthernet3/0
48.73.239.11      1   FULL/DR         00:00:39    10.1.3.1        GigabitEthernet2/0
48.73.239.44      1   FULL/DR         00:00:38    10.3.4.2        GigabitEthernet1/0

R3#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.3.3.3/32        1     LOOP  0/0
Gi3/0        1     0               10.3.5.1/30        1     BDR   1/1
Gi2/0        1     0               10.1.3.2/30        1     BDR   1/1
Gi1/0        1     0               10.3.4.1/30        1     BDR   1/1
```

```txt 
! ROUTER 4 --------------------------------------------------------------
R4#sh ip ospf database

            OSPF Router with ID (48.73.239.44) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    254         0x8000000B 0x004294 3
48.73.239.22    48.73.239.22    713         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    212         0x8000000A 0x008DD7 4
48.73.239.44    48.73.239.44    208         0x8000000A 0x003111 4
48.73.239.55    48.73.239.55    203         0x80000009 0x006C2B 2
48.73.239.66    48.73.239.66    1764        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    713         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    254         0x80000006 0x00F5BF
10.2.4.2        48.73.239.44    1982        0x80000004 0x00C2BA
10.3.4.2        48.73.239.44    208         0x80000008 0x004924
10.3.5.2        48.73.239.55    203         0x80000008 0x006AEB
10.4.6.2        48.73.239.66    1764        0x80000004 0x002215

R4#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.2.2.2/32 [110/2] via 10.2.4.1, 02:09:24, GigabitEthernet2/0
O       10.1.3.0/30 [110/2] via 10.3.4.1, 01:44:06, GigabitEthernet1/0
O       10.3.3.3/32 [110/2] via 10.3.4.1, 01:44:06, GigabitEthernet1/0
O       10.1.2.0/30 [110/2] via 10.2.4.1, 02:09:24, GigabitEthernet2/0
O       10.1.1.1/32 [110/3] via 10.3.4.1, 01:44:06, GigabitEthernet1/0
                    [110/3] via 10.2.4.1, 01:53:06, GigabitEthernet2/0
O       10.6.6.6/32 [110/2] via 10.4.6.2, 02:09:34, GigabitEthernet3/0
O       10.3.5.0/30 [110/2] via 10.3.4.1, 01:44:06, GigabitEthernet1/0
O       10.5.5.5/32 [110/3] via 10.3.4.1, 01:44:06, GigabitEthernet1/0

R4#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.66      1   FULL/DR         00:00:37    10.4.6.2        GigabitEthernet3/0
48.73.239.22      1   FULL/BDR        00:00:33    10.2.4.1        GigabitEthernet2/0
48.73.239.33      1   FULL/BDR        00:00:32    10.3.4.1        GigabitEthernet1/0

R4#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.4.4.4/32        1     LOOP  0/0
Gi3/0        1     0               10.4.6.1/30        1     BDR   1/1
Gi2/0        1     0               10.2.4.2/30        1     DR    1/1
Gi1/0        1     0               10.3.4.2/30        1     DR    1/1
```

```txt
! ROUTER 5 --------------------------------------------------------------
R5#show ip ospf database

            OSPF Router with ID (48.73.239.55) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    342         0x8000000B 0x004294 3
48.73.239.22    48.73.239.22    803         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    300         0x8000000A 0x008DD7 4
48.73.239.44    48.73.239.44    298         0x8000000A 0x003111 4
48.73.239.55    48.73.239.55    289         0x80000009 0x006C2B 2
48.73.239.66    48.73.239.66    1853        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    803         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    342         0x80000006 0x00F5BF
10.2.4.2        48.73.239.44    41          0x80000005 0x00C0BB
10.3.4.2        48.73.239.44    298         0x80000008 0x004924
10.3.5.2        48.73.239.55    289         0x80000008 0x006AEB
10.4.6.2        48.73.239.66    1853        0x80000004 0x002215

R5#show ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/3] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.2.2.2/32 [110/4] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.1.3.0/30 [110/2] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.3.3.3/32 [110/2] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.1.2.0/30 [110/3] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.1.1.1/32 [110/3] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.6.6.6/32 [110/4] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.2.4.0/30 [110/3] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.3.4.0/30 [110/2] via 10.3.5.1, 01:45:23, GigabitEthernet3/0
O       10.4.4.4/32 [110/3] via 10.3.5.1, 01:45:23, GigabitEthernet3/0

R5#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.33      1   FULL/BDR        00:00:34    10.3.5.1        GigabitEthernet3/0

R5#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.5.5.5/32        1     LOOP  0/0
Gi3/0        1     0               10.3.5.2/30        1     DR    1/1
```

```txt
! ROUTER 6 --------------------------------------------------------------
R6#show ip ospf database

            OSPF Router with ID (48.73.239.66) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
48.73.239.11    48.73.239.11    419         0x8000000B 0x004294 3
48.73.239.22    48.73.239.22    879         0x80000008 0x004971 3
48.73.239.33    48.73.239.33    377         0x8000000A 0x008DD7 4
48.73.239.44    48.73.239.44    374         0x8000000A 0x003111 4
48.73.239.55    48.73.239.55    368         0x80000009 0x006C2B 2
48.73.239.66    48.73.239.66    1927        0x80000005 0x003945 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        48.73.239.22    879         0x80000006 0x00EDC7
10.1.3.1        48.73.239.11    419         0x80000006 0x00F5BF
10.2.4.2        48.73.239.44    117         0x80000005 0x00C0BB
10.3.4.2        48.73.239.44    374         0x80000008 0x004924
10.3.5.2        48.73.239.55    368         0x80000008 0x006AEB
10.4.6.2        48.73.239.66    1927        0x80000004 0x002215

R6#show ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.2.2.2/32 [110/3] via 10.4.6.1, 02:11:58, GigabitEthernet3/0
O       10.1.3.0/30 [110/3] via 10.4.6.1, 01:46:40, GigabitEthernet3/0
O       10.3.3.3/32 [110/3] via 10.4.6.1, 01:46:40, GigabitEthernet3/0
O       10.1.2.0/30 [110/3] via 10.4.6.1, 02:11:58, GigabitEthernet3/0
O       10.1.1.1/32 [110/4] via 10.4.6.1, 01:55:40, GigabitEthernet3/0
O       10.3.5.0/30 [110/3] via 10.4.6.1, 01:46:40, GigabitEthernet3/0
O       10.2.4.0/30 [110/2] via 10.4.6.1, 02:12:08, GigabitEthernet3/0
O       10.3.4.0/30 [110/2] via 10.4.6.1, 01:46:40, GigabitEthernet3/0
O       10.4.4.4/32 [110/2] via 10.4.6.1, 02:12:08, GigabitEthernet3/0
O       10.5.5.5/32 [110/4] via 10.4.6.1, 01:46:40, GigabitEthernet3/0

R6#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
48.73.239.44      1   FULL/BDR        00:00:31    10.4.6.1        GigabitEthernet3/0

R6#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.6.6.6/32        1     LOOP  0/0
Gi3/0        1     0               10.4.6.2/30        1     DR    1/1
```

__AS 17390__ - IBM

```txt
! From Router 7 - ping Router 8 Loopback 0 ------------------------------
R7#ping 10.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/21/36 ms
```

OSPF configuration analysis

```txt
! ROUTER 7 --------------------------------------------------------------
R7#show ip ospf database

            OSPF Router with ID (130.41.46.77) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
130.41.46.77    130.41.46.77    1385        0x80000005 0x007902 2
130.41.46.88    130.41.46.88    1347        0x80000005 0x00FD63 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.7.8.2        130.41.46.88    1347        0x80000004 0x00197A

R7#show ip route ospf
     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O       10.8.8.8/32 [110/2] via 10.7.8.2, 02:03:15, GigabitEthernet1/0

R7#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
130.41.46.88      1   FULL/DR         00:00:36    10.7.8.2        GigabitEthernet1/0

R7#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.7.7.7/32        1     LOOP  0/0
Gi1/0        1     0               10.7.8.1/30        1     BDR   1/1
```

```txt
! ROUTER 8 --------------------------------------------------------------
R8#show ip ospf database

            OSPF Router with ID (130.41.46.88) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
130.41.46.77    130.41.46.77    400         0x80000006 0x007703 2
130.41.46.88    130.41.46.88    358         0x80000006 0x00FB64 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.7.8.2        130.41.46.88    358         0x80000005 0x00177B

R8#show ip route ospf
     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O       10.7.7.7/32 [110/2] via 10.7.8.1, 02:19:58, GigabitEthernet1/0

R8#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
130.41.46.77      1   FULL/BDR        00:00:37    10.7.8.1        GigabitEthernet1/0

R8#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.8.8.8/32        1     LOOP  0/0
Gi1/0        1     0               10.7.8.2/30        1     DR    1/1
```

__AS 5511__ - FTRSI (Orange - Worldwide IP Backbone)

```txt
From Router 10 - ping Routers 11 and 12 Loopback 0 ----------------------
R10#ping 10.11.11.11

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.11.11.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/20/44 ms
R10#ping 10.12.12.12

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.12.12.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/32/80 ms
```

OSPF configuration analysis

```txt
! ROUTER 10 -------------------------------------------------------------
R10#show ip ospf database

            OSPF Router with ID (10.10.10.10) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     321         0x8000000B 0x00421C 2
10.11.11.11     10.11.11.11     401         0x8000000B 0x004113 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.11.1      10.10.10.10     321         0x80000005 0x004C51

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.0      10.10.10.10     1090        0x80000006 0x00CA27
10.10.12.0      10.11.11.11     401         0x80000003 0x00C52B
10.11.12.0      10.10.10.10     582         0x80000003 0x00CE24
10.11.12.0      10.11.11.11     401         0x80000007 0x00A745
10.12.12.12     10.10.10.10     582         0x80000003 0x005C86
10.12.12.12     10.11.11.11     401         0x80000003 0x004798
46.87.162.0     10.10.10.10     582         0x80000003 0x00ECFE
46.87.162.0     10.11.11.11     403         0x80000003 0x00D711
46.87.162.112   10.10.10.10     584         0x80000003 0x009ADD
46.87.162.112   10.11.11.11     403         0x80000003 0x0085EF

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     584         0x80000009 0x00A0F8 1
10.11.11.11     10.11.11.11     404         0x80000009 0x008808 1
10.12.12.12     10.12.12.12     528         0x8000000D 0x006CA7 5

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.1      10.10.10.10     584         0x80000003 0x008A13
10.11.12.2      10.12.12.12     528         0x80000005 0x00553B

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     1091        0x80000006 0x007A96
0.0.0.0         10.11.11.11     404         0x80000007 0x0063A9
10.10.10.10     10.10.10.10     1092        0x80000006 0x00AC3C
10.10.10.10     10.11.11.11     406         0x80000003 0x00A740
10.10.11.0      10.10.10.10     1094        0x80000006 0x00F301
10.10.11.0      10.11.11.11     406         0x80000008 0x00DA15
10.11.11.11     10.10.10.10     325         0x8000000C 0x008955
10.11.11.11     10.11.11.11     406         0x80000008 0x00726E

R10#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O       10.11.11.11/32 [110/2] via 10.10.11.2, 01:13:38, GigabitEthernet1/0
O       10.12.12.12/32 [110/2] via 10.10.12.2, 01:16:04, GigabitEthernet3/0
O       10.11.12.0/30 [110/2] via 10.10.12.2, 01:16:04, GigabitEthernet3/0
     46.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
O       46.87.162.112/32 [110/2] via 10.10.12.2, 01:16:04, GigabitEthernet3/0
O       46.87.162.0/30 [110/2] via 10.10.12.2, 01:16:04, GigabitEthernet3/0

R10#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.11.11.11       1   FULL/BDR        00:00:33    10.10.11.2      GigabitEthernet1/0
10.12.12.12       1   FULL/BDR        00:00:36    10.10.12.2      GigabitEthernet3/0

R10#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.10.10.10/32     1     LOOP  0/0
Gi1/0        1     0               10.10.11.1/30      1     DR    1/1
Gi3/0        1     1               10.10.12.1/30      1     DR    1/1
```

```txt
! ROUTER 11 -------------------------------------------------------------
R11#show ip ospf database

            OSPF Router with ID (10.11.11.11) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     461         0x8000000B 0x00421C 2
10.11.11.11     10.11.11.11     539         0x8000000B 0x004113 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.11.1      10.10.10.10     461         0x80000005 0x004C51

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.0      10.10.10.10     1230        0x80000006 0x00CA27
10.10.12.0      10.11.11.11     539         0x80000003 0x00C52B
10.11.12.0      10.10.10.10     722         0x80000003 0x00CE24
10.11.12.0      10.11.11.11     539         0x80000007 0x00A745
10.12.12.12     10.10.10.10     722         0x80000003 0x005C86
10.12.12.12     10.11.11.11     539         0x80000003 0x004798
46.87.162.0     10.10.10.10     722         0x80000003 0x00ECFE
46.87.162.0     10.11.11.11     541         0x80000003 0x00D711
46.87.162.112   10.10.10.10     723         0x80000003 0x009ADD
46.87.162.112   10.11.11.11     541         0x80000003 0x0085EF

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     724         0x80000009 0x00A0F8 1
10.11.11.11     10.11.11.11     541         0x80000009 0x008808 1
10.12.12.12     10.12.12.12     667         0x8000000D 0x006CA7 5

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.1      10.10.10.10     724         0x80000003 0x008A13
10.11.12.2      10.12.12.12     667         0x80000005 0x00553B

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     1232        0x80000006 0x007A96
0.0.0.0         10.11.11.11     541         0x80000007 0x0063A9
10.10.10.10     10.10.10.10     1232        0x80000006 0x00AC3C
10.10.10.10     10.11.11.11     541         0x80000003 0x00A740
10.10.11.0      10.10.10.10     1233        0x80000006 0x00F301
10.10.11.0      10.11.11.11     541         0x80000008 0x00DA15
10.11.11.11     10.10.10.10     464         0x8000000C 0x008955
10.11.11.11     10.11.11.11     541         0x80000008 0x00726E

R11#show ip route ospf
     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O       10.10.10.10/32 [110/2] via 10.10.11.1, 01:16:03, GigabitEthernet1/0
O       10.12.12.12/32 [110/2] via 10.11.12.2, 01:16:03, GigabitEthernet2/0
O       10.10.12.0/30 [110/2] via 10.11.12.2, 01:16:03, GigabitEthernet2/0
     46.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
O       46.87.162.112/32 [110/2] via 10.11.12.2, 01:16:03, GigabitEthernet2/0
O       46.87.162.0/30 [110/2] via 10.11.12.2, 01:16:03, GigabitEthernet2/0

R11#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.10.10       1   FULL/DR         00:00:36    10.10.11.1      GigabitEthernet1/0
10.12.12.12       1   FULL/DR         00:00:38    10.11.12.2      GigabitEthernet2/0

R11#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.11.11.11/32     1     LOOP  0/0
Gi1/0        1     0               10.10.11.2/30      1     BDR   1/1
Gi2/0        1     1               10.11.12.1/30      1     BDR   1/1
```

```txt
! ROUTER 12 -------------------------------------------------------------
R12#show ip ospf database

            OSPF Router with ID (10.12.12.12) (Process ID 1)

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     943         0x80000009 0x00A0F8 1
10.11.11.11     10.11.11.11     761         0x80000009 0x008808 1
10.12.12.12     10.12.12.12     886         0x8000000D 0x006CA7 5

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.1      10.10.10.10     943         0x80000003 0x008A13
10.11.12.2      10.12.12.12     886         0x80000005 0x00553B

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     1451        0x80000006 0x007A96
0.0.0.0         10.11.11.11     761         0x80000007 0x0063A9
10.10.10.10     10.10.10.10     1451        0x80000006 0x00AC3C
10.10.10.10     10.11.11.11     761         0x80000003 0x00A740
10.10.11.0      10.10.10.10     1451        0x80000006 0x00F301
10.10.11.0      10.11.11.11     763         0x80000008 0x00DA15
10.11.11.11     10.10.10.10     683         0x8000000C 0x008955
10.11.11.11     10.11.11.11     763         0x80000008 0x00726E

R12#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA    10.10.10.10/32 [110/2] via 10.10.12.1, 01:21:57, GigabitEthernet3/0
O IA    10.11.11.11/32 [110/2] via 10.11.12.1, 01:19:31, GigabitEthernet2/0
O IA    10.10.11.0/30 [110/2] via 10.11.12.1, 01:19:31, GigabitEthernet2/0
                      [110/2] via 10.10.12.1, 01:21:57, GigabitEthernet3/0
O*IA 0.0.0.0/0 [110/2] via 10.11.12.1, 01:19:31, GigabitEthernet2/0
               [110/2] via 10.10.12.1, 01:21:57, GigabitEthernet3/0

R12#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.10.10       1   FULL/DR         00:00:33    10.10.12.1      GigabitEthernet3/0
10.11.11.11       1   FULL/BDR        00:00:37    10.11.12.1      GigabitEthernet2/0

R12#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     1               10.12.12.12/32     1     LOOP  0/0
Lo1          1     1               46.87.162.112/32   1     LOOP  0/0
Fa0/0        1     1               46.87.162.2/30     1     DR    0/0
Gi3/0        1     1               10.10.12.2/30      1     BDR   1/1
Gi2/0        1     1               10.11.12.2/30      1     DR    1/1
```

#### Test the interfaces connectivity between the ASes

__Example from AS 1273 to AS 17390__

| From | To | Interface | IP Address | Connectivity Success |
| ---- | -- | --------- | ---------- | ------- |
| R1   | R8 | Lo1 | 130.41.46.88     | No      |
| R1   | R8 | g4/0 | 48.73.240.2     | No     |  
| R1   | R8 | g5/0 | 130.41.46.2     | No      |
| R1   | R7 | Lo1  | 130.41.46.77    | No |
| R1   | R7 | f0/0 | 130.41.46.9     | No |
| R1   | R7 | g4/0 | 48.73.240.6     | No |
| R1   | R7 | g5/0 | 130.41.46.2     | No |

We have no connectivity between ASes, even in direct connected links. To have connectivity between ASes we need to configure BGP.

### 1.4 Practical Questions

#### 1.4.1 - Create a comprehensive table presenting all the connectivity tests carried out and the respective outcome (e.g., success, failure). You don’t need to provide exhaustive snapshots for all test results. Choose only **three example cases**, from different ASes, to include in your report and briefly comment on each selected case.

__TODO__
pings do R1 para os routers do mesmo AS e para R7, R8
pings do AS com area 0 e area 1
pings do AS que tem um AS privado

#### 1.4.2 - Identification of the ASes entities involved in the lab topology

| ASN   | AS Name                                | Country   |
| ----- | -------------------------------------- | --------- |
|  701  | Verizon Business                       | US        |
| 64513 | (Private AS)                           |           |
| 17390 | IBM                                    | US        |
| 4637  | Telstra Global (APNIC)                 | Hong Kong |
| 1273  | Vodafone                               | UK        |
| 1     | Level 3 Parent, LLC                    | US        |
| 20717 | DE-CIX Management GmbH                 | Germany   |
| 5511  | FTRSI (Orange - Worldwide IP Backbone) | France    |
| 23344 | Disney Worldwide Services, Inc.        | US        |

#### 1.4.3 - Explain the concept of an Autonomous System (AS) in the Internet architecture and provide examples associated with the Portuguese Internet ecosystem

In the Internet architecture, an Autonomous System (AS) is a large network or group of networks controlled by a single organization that operates under a unified routing policy. Each AS is assigned a unique Autonomous System Number (ASN) for identification when exchanging traffic with other networks.

Portugal's Internet infrastructure consists of numerous independent networks. There 123 ASNs allocated to Portugal and are managed by various Portuguese ISPs, telecommunications companies, universities, government agencies, banks, or other large enterprises.

[List of Portuguese ASes](https://ipfind.io/autonomous-system/countries/Portugal)

#### 1.4.4 - Classification of each AS in the lab project as Tier-1 or Tier-2. For each classification, describe the evidence you find in the lab topology that justifies it

__TODO__: check and discuss


### Classification of Autonomous Systems

| AS Number | AS Name / Entity | Classification | Evidence from Lab Topology |
| :--- | :--- | :--- | :--- |
| **AS701** | Verizon Business | **Tier-1** | Described as a "Tier 1" AS in the project overview (Section 3.2). It peers with multiple other major ASes (AS1273, AS17390) and does not need to purchase transit from anyone, fitting the definition of a top-level provider. |
| **AS1** | Level 3 Parent, LLC | **Tier-1** | A historically well-known Tier-1 ISP. The topology shows it is directly connected to other major networks (AS4637), and it is mentioned as part of the "Tier 1 and Tier 2" ecosystem. |
| **AS1273** | Vodafone | **Tier-2** | A large telecommunications provider. While global, it acts as a customer of AS701 (purchasing transit), which is a key indicator of a Tier-2. It also peers with multiple other ASes (AS17390, AS20717, AS5511) to reduce its transit costs. |
| **AS4637** | Telstra Global | **Tier-2** | A dominant national provider in Australia, but on the global stage, it functions as a Tier-2. The topology shows it connecting to other major ASes (AS1, AS20717, AS23344), engaging in a mix of peering and transit relationships. |
| **AS5511** | Orange (FTRSI) | **Tier-2** | Another major international telecom operator. Its role in the lab is that of a large provider that peers with others (e.g., via AS20717) but is not depicted as part of the exclusive Tier-1 core. |
| **AS17390** | IBM | **Enterprise (Tier-3)** | This is a corporate enterprise network. Its purpose is to connect its own services (like IBM Cloud) to the internet, not to sell transit to others. It purchases transit from AS1273, making it a classic Tier-3 / "stub" AS. |
| **AS23344** | Disney | **Enterprise (Tier-3)** | This is a content/enterprise network for Disney. It has only one eBGP peering to the internet (with AS4637) and is configured to receive only a default route, which is typical behavior for an enterprise customer. |
| **AS20717** | DE-CIX Management GmbH | **Internet Exchange (IXP)** | This is not a transit provider. It is the AS for the DE-CIX Internet Exchange Point. Its function is to provide a neutral fabric for other ASes to peer, which places it outside the Tier hierarchy. |
| **AS64513** | (Private AS) | **Unclassifiable** | This is a Private AS number (range 64512-65534) used internally, likely for a multihomed customer within AS17390. It is not advertised to the global internet and has no tier classification. |

---

### Justification and Analysis

The classification is based on the core differentiator between tiers: **transit and peering relationships**.

1.  **Tier-1 Evidence:** AS701 and AS1 are presented as the foundational providers in the topology. A key piece of evidence is that **AS1273 (Vodafone) purchases transit from AS701**. The fact that one AS is a customer of another immediately defines the provider (AS701) as higher in the hierarchy. Tier-1 networks form a club that can reach the entire internet via settlement-free peering with each other, and they sell transit to all others.

2.  **Tier-2 Evidence:** AS1273, AS4637, and AS5511 are all massive telecommunications companies. However, their behavior in the lab aligns with Tier-2s:
    *   They operate large backbones and provide transit to their own customers (e.g., AS1273 provides transit to AS17390).
    *   Crucially, they also **purchase transit** from Tier-1s (as seen with AS1273 and AS701).
    *   They engage in extensive **peering** at neutral points (like AS20717) to exchange traffic directly with other networks, reducing their reliance on expensive Tier-1 transit links. This mixed strategy of buying transit and peering for free is the hallmark of a Tier-2.

3.  **Enterprise (Tier-3) Evidence:** AS17390 (IBM) and AS23344 (Disney) are "leaf" networks. Their primary goal is to have their services reachable on the internet. They do not sell transit to any other AS. Instead, they are customers of larger ISPs. Disney's requirement to receive "only the default route" from its provider is a common configuration for enterprise networks that do not want to hold the full internet routing table.

4.  **Special Case - IXP:** AS20717 (DE-CIX) is not a tiered AS. It is infrastructure. It does not buy or sell transit. It exists so that ASes from all tiers (including the Tier-1s, Tier-2s, and Enterprises in this lab) can connect and exchange traffic efficiently.

#### 1.4.5 - Table showcasing all the peering relations established in the provided topology

| AS | Peers |
| --- | --- |
| 1273 | 20717, 4637, 701, 17390, 5511 |
| 17390 | 701, 1273 |
| 701 | 17390, 1273, 4637 |
| 4637 | 701, 1273, 1, 5511 |
| 1 | 4637 |
| 20717 | 1273, 5511 |
| 5511 | 20717, 23344, 1273, 4537 |
| 23344 | 5511 |

#### 1.4.6 - Explain how a Tier-2 benefits from peering instead of buying everything from a Tier-1

A Tier-2 ISP operates on a simple principle: it must pay a Tier-1 for access to the entire internet. However, a significant portion of its traffic is often destined for a handful of other large networks (like other Tier-2s, cloud providers, or content giants like Google and Netflix).

Peering is the strategy of connecting directly with these specific networks to exchange traffic, creating a "shortcut" that bypasses the Tier-1 transit path.

Key Benefits of Peering for a Tier-2:

- Cost Reduction
- Improved Performance and Lower Latency
- Increased Reliability and Redundancy
- Greater Control over Routing

#### 1.4.7 - Identify the neutral public peering interconnections in this lab topology. Elaborate on why they are called neutral and provide examples of real-world implementations of such public interconnections

The neutral public peering interconnection is represented by AS20717 - DE-CIX Management GmbH.

Instead of setting up direct, individual links between each other (private peering), these ASes can all connect to the shared switch fabric of AS20717

Examples of real world implementations:

- DE-CIX (Frankfurt) - de-cix.net (lab)
- AMS-IX (Amsterdam) - ams-ix.net
- LINX (London) - linx.net
- EQUINIX IX (Various) - equinix.com/ix
- NYIIX (New York) - nyiix.net

By connecting to these neutral hubs, the Tier-2 ISPs can exchange a large volume of their traffic directly, locally, and for free, achieving the significant benefits of peering.

#### 1.4.8 - Explain the role of R12 in AS 5511, and how are its interfaces divided between the OSPF areas involved

R12 serves as an Area Border Router (ABR). This is its primary and most important role.

As an ABR, R12 has the following key responsibilities:

Connecting Different OSPF Areas: It has interfaces in two separate OSPF areas: the backbone area (Area 0) and a stub area (Area 1). It is the sole router providing a pathway between these two parts of the network.

Summarizing and Filtering LSAs: It controls the flow of Link-State Advertisements (LSAs) between Area 0 and Area 1.

Injecting a Default Route: As Area 1 is a stub area, R12 does not flood external LSAs (Type 5 LSAs) into it. Instead, it injects a default route (0.0.0.0/0) into Area 1, telling all routers in that area, "if you don't have a more specific route, send the traffic to me to reach external destinations."

#### 1.4.9 - Explain what a stub area is and discuss the resulting advantages and potential limitations. In your discussion, please detail under what conditions would multi-area OSPF be preferred over a single backbone area in real networks

In the OSPF (Open Shortest Path First) routing protocol, the network is divided into **areas** to create a hierarchical structure. This hierarchy is crucial for controlling the flow of routing information and improving scalability.

A **stub area** is a specific type of OSPF area designed to minimize the size of the routing table and the amount of Link-State Advertisement (LSA) traffic flooding into it.

**Core Concept:** A stub area is configured to block external routes from entering. These external routes are routes redistributed into OSPF from other routing protocols (like BGP, EIGRP, or static routes). They are carried by **Type 5 LSAs**.

Instead of learning these potentially vast numbers of external routes, a router in a stub area uses a **default route (0.0.0.0/0)**, injected by the Area Border Router (ABR), to reach any destination outside the OSPF domain.

**Key Characteristics of a Stub Area:**
*   **Blocks Type 5 LSAs:** The ABR does not flood AS External LSAs (Type 5) into the stub area.
*   **Uses a Default Route:** The ABR automatically generates and advertises a default route (Type 3 LSA) into the stub area.
*   **Cannot be a Transit Area:** Traffic from one non-backbone area cannot use a stub area to reach another non-backbone area. Virtual Links cannot be configured through a stub area.
*   **No ASBRs Allowed:** An Autonomous System Boundary Router (ASBR) cannot be located inside a stub area, as its job is to generate the very Type 5 LSAs that are blocked.

There are more specific types of stub areas like **Totally Stubby Areas** (which also block inter-area routes, Type 3 LSAs) and **NSSA (Not-So-Stubby Area)** which allows for limited external route import, but the classic stub area is the foundational concept.

__Advantages__

1.  **Reduced Routing Table Size:** This is the primary benefit. By replacing thousands of external routes with a single default route, the routing tables of routers within the stub area are significantly smaller. This saves memory (RAM).
2.  **Reduced LSA Traffic and CPU Overload:** By blocking Type 5 LSAs, there is less Link-State Update traffic flooding within the area. This reduces the CPU cycles required for OSPF SPF (Shortest Path First) calculations, especially when external networks are flapping.
3.  **Improved Stability:** Instabilities in external networks (e.g., Internet route changes) are completely isolated from the routers inside the stub area. This makes the internal OSPF domain more stable.
4.  **Simpler Configuration and Management:** For remote branch offices that only need a default path to the central data center/Internet, a stub area configuration is simple and logical.

__Potential Limitations__

1.  **Suboptimal Routing:** This is the most significant trade-off. Because internal routers only have a default route, they have no topological awareness of external paths. All traffic destined outside the OSPF domain is sent to the ABR, which may not be the most efficient path if multiple exits exist.
2.  **Loss of Granular Control:** Network administrators lose the ability to make precise routing decisions for external destinations within the stub area, as the specific routes are unknown.
3.  **Restrictive Topology:** The requirement that no ASBR can reside in a stub area limits network design flexibility. You cannot have a branch office that also connects to, for example, a partner network via BGP if it's configured as a stub area (you would use an NSSA instead).

#### 1.4.10 - Discuss why the subnet 46.87.162.0/24 was not placed on the backbone area, considering the OSPF design principles

The fundamental rule is: "Keep the backbone clean." Area 0 should be a high-speed transit core for inter-area traffic, not a place for end-user or server subnets.

Key Reasons for Stub Area Placement:

- Hierarchy & Stability
- Reduced Routing Overhead
- Controlled Traffic Flow
- Scalability

Putting end-user subnets in Area 0 violates OSPF hierarchy, unnecessarily exposes the core to edge network instability, and eliminates the benefits of LSA filtering, leading to a less scalable and resilient network.

<div style="page-break-after: always"></div>

## 2 - Phase 2 - iBGP and eBGP Without Routing Policies

### 2.1 - Objectives

In this phase we have as objectives:

- Establish eBGP sessions between the AS’s
- Inside the AS 1273, establish iBGP sessions between the clients and the two route reflectors, R3 and R4 to avoid the full mesh
- Server subnet public IPs are listed in the internet routing table
- Implement connectivity between the ASes from any routers using the Lo1 and from any server

### 2.2 - Implementation

#### BGP Configuration

__AS 1273__ - Vodafone

```txt
! Router 1 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
=============================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
!
! =============================================================================
!                                   BGP
! =============================================================================
!
router bgp 1273
 bgp router-id 10.1.1.1
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP - adjacencies in Loopback interfaces -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor 10.3.3.3 peer-group iBGP
 neighbor 10.4.4.4 peer-group iBGP
 !
 ! eBGP - adjacencies in phisical interfaces --------------------------------
 neighbor 48.73.240.2 remote-as 17390
 neighbor 48.73.240.2 soft-reconfiguration inbound
 !
 ! Advertise Routes ---------------------------------------------------------
 network 48.73.239.11 mask 255.255.255.255     ! Loopback 1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.240.0 mask 255.255.252.0
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary                               ! Do not summarize routes 
```

```txt
! Router 2 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1273
 bgp router-id 10.2.2.2
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor 10.3.3.3 peer-group iBGP
 neighbor 10.4.4.4 peer-group iBGP
 !
 ! Advertise Routes -------------------
 network 48.73.239.22 mask 255.255.255.255       ! Lo1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.239.0 mask 255.255.255.252        ! avoid black-hole in access to server2
 network 48.73.240.0 mask 255.255.252.0
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary
```

```txt
! Router 3 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
! 
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1273
 bgp router-id 10.3.3.3
 bgp cluster-id 1273
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP route-reflector-client
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor iBGP_RR peer-group
 neighbor iBGP_RR remote-as 1273
 neighbor iBGP_RR update-source Loopback0
 neighbor iBGP_RR soft-reconfiguration inbound
 !
 neighbor 10.1.1.1 peer-group iBGP
 neighbor 10.2.2.2 peer-group iBGP
 neighbor 10.4.4.4 peer-group iBGP_RR
 neighbor 10.5.5.5 peer-group iBGP
 neighbor 10.6.6.6 peer-group iBGP
 !
 ! eBGP -------------------------------
 neighbor 48.73.240.6 remote-as 17390
 neighbor 48.73.240.6 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 48.73.239.33 mask 255.255.255.255   ! Lo1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.240.0 mask 255.255.252.0
 network 48.73.240.4 mask 255.255.255.252    ! to AS 17390 avoiding black-hole
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary
```

```txt
! Router 4 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
! 
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1273
 bgp router-id 10.4.4.4
 bgp cluster-id 1273
 no synchronization
 bgp log-neighbor-changes
 ! iBGP -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP route-reflector-client
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor iBGP_RR peer-group
 neighbor iBGP_RR remote-as 1273
 neighbor iBGP_RR update-source Loopback0
 neighbor iBGP_RR soft-reconfiguration inbound
 !
 neighbor 10.1.1.1 peer-group iBGP
 neighbor 10.2.2.2 peer-group iBGP
 neighbor 10.3.3.3 peer-group iBGP_RR
 neighbor 10.5.5.5 peer-group iBGP
 neighbor 10.6.6.6 peer-group iBGP
 !
 ! Advertise Routes -------------------
 network 48.73.239.44 mask 255.255.255.255    ! Lo1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.240.0 mask 255.255.252.0
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary
```

```txt
! Router 5 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
! 
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1273
 bgp router-id 10.5.5.5
 no synchronization
 bgp log-neighbor-changes
 ! iBGP -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor 10.3.3.3 peer-group iBGP
 neighbor 10.4.4.4 peer-group iBGP
 !
 ! eBGP -------------------------------
 neighbor 64.112.0.1 remote-as 701
 neighbor 64.112.0.1 soft-reconfiguration-inbound
 !
 ! Advertise Routes -------------------
 network 48.73.239.55 mask 255.255.255.255   ! Lo1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.240.0 mask 255.255.252.0
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary
```

```txt
! Router 6 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 48.73.239.0 255.255.255.0 Null0
ip route 48.73.240.0 255.255.252.0 Null0
ip route 63.25.64.0 255.255.192.0 Null0
! 
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1273
 bgp router-id 10.6.6.6
 no synchronization
 bgp log-neighbor-changes
 ! iBGP -------------------------------
 neighbor iBGP peer-group
 neighbor iBGP remote-as 1273
 neighbor iBGP update-source Loopback0
 neighbor iBGP next-hop-self
 neighbor iBGP soft-reconfiguration inbound
 !
 neighbor 10.3.3.3 peer-group iBGP
 neighbor 10.4.4.4 peer-group iBGP
 !
 ! eBGP -------------------------------
 neighbor 48.73.240.14 remote-as 4637
 neighbor 48.73.240.14 soft-reconfiguration inbound
 neighbor 48.73.240.22 remote-as 20717
 neighbor 48.73.240.22 soft-reconfiguration inbound
 neighbor 48.73.240.18 remote-as 5511
 neighbor 48.73.240.18 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 48.73.239.66 mask 255.255.255.255   ! Lo1
 network 48.73.239.0 mask 255.255.255.0
 network 48.73.240.0 mask 255.255.252.0
 network 63.25.64.0 mask 255.255.192.0
 no auto-summary
```

__AS 17390__ - IBM

```txt
! Router 7 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 130.41.46.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 17390
 bgp router-id 10.7.7.7
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor 10.8.8.8 remote-as 17390
 neighbor 10.8.8.8 update-source Loopback0
 neighbor 10.8.8.8 next-hop-self
 neighbor 10.8.8.8 soft-reconfiguration inbound
 !
 ! eBGP -------------------------------
 neighbor 64.112.0.5 remote-as 701
 neighbor 64.112.0.5 soft-reconfiguration inbound
 neighbor 130.41.46.10 remote-as 64513
 neighbor 130.41.46.10 soft-reconfiguration inbound
 neighbor 48.73.240.5 remote-as 1273
 neighbor 48.73.240.5 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 130.41.46.77 mask 255.255.255.255     ! Lo1
 network 130.41.46.0 mask 255.255.255.0
 no auto-summary
```

```txt
! Router 8 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 130.41.46.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 17390
 bgp router-id 10.8.8.8
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor 10.7.7.7 remote-as 17390
 neighbor 10.7.7.7 update-source Loopback0
 neighbor 10.7.7.7 next-hop-self
 neighbor 10.7.7.7 soft-reconfiguration inbound
 !
 ! eBGP -------------------------------
 neighbor 130.41.46.6 remote-as 64513
 neighbor 130.41.46.6 soft-reconfiguration inbound
 neighbor 48.73.240.1 remote-as 1273
 neighbor 48.73.240.1 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 130.41.46.0 mask 255.255.255.0
 network 130.41.46.0 mask 255.255.255.252   ! avoid black hole / access server1
 network 130.41.46.88 mask 255.255.255.255  ! Loopback1
 no auto-summary
```

__AS 64513__ - Private (inside IBM)

```txt
! Router 9 ==================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 130.41.47.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 64513
 bgp router-id 10.9.9.9
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 130.41.46.9 remote-as 17390
 neighbor 130.41.46.9 soft-reconfiguration inbound
 neighbor 130.41.46.5 remote-as 17390
 neighbor 130.41.46.5 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 130.41.47.99 mask 255.255.255.255    ! Lo1
 network 130.41.47.0 mask 255.255.255.0
 no auto-summary
```

__AS 5511__ - Orange

```txt
! Router 10 =================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 46.87.162.0 255.255.255.0 Null0
ip route 46.88.20.0 255.255.255.0 Null0
ip route 46.88.22.0 255.255.255.0 Null0
ip route 46.89.36.0 255.255.255.0 Null0
ip route 46.89.38.0 255.255.255.0 Null0
ip route 46.92.2.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
!
router bgp 5511
 bgp router-id 10.10.10.10
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor 10.11.11.11 remote-as 5511
 neighbor 10.11.11.11 update-source Loopback0
 neighbor 10.11.11.11 next-hop-self
 neighbor 10.11.11.11 soft-reconfiguration inbound
 neighbor 10.12.12.12 remote-as 5511
 neighbor 10.12.12.12 update-source Loopback0
 neighbor 10.12.12.12 next-hop-self
 neighbor 10.12.12.12 soft-reconfiguration inbound
 !
 ! eBGP -------------------------------
 neighbor 46.88.20.2 remote-as 20717
 neighbor 46.88.20.2 soft-reconfiguration inbound
 neighbor 48.73.240.17 remote-as 1273
 neighbor 48.73.240.17 soft-reconfiguration inbound
 neighbor 211.176.129.1 remote-as 4637
 neighbor 211.176.129.1 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 46.87.162.110 mask 255.255.255.255     ! Lo1
 network 46.87.162.0 mask 255.255.255.0
 network 46.88.20.0 mask 255.255.255.0
 network 46.88.22.0 mask 255.255.255.0
 network 46.89.36.0 mask 255.255.255.0
 network 46.89.38.0 mask 255.255.255.0
 network 46.92.2.0 mask 255.255.255.0
 no auto-summary
```

```txt
! Router 11 =================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 46.87.162.0 255.255.255.0 Null0
ip route 46.88.20.0 255.255.255.0 Null0
ip route 46.88.22.0 255.255.255.0 Null0
ip route 46.89.36.0 255.255.255.0 Null0
ip route 46.89.38.0 255.255.255.0 Null0
ip route 46.92.2.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 5511
 bgp router-id 10.11.11.11
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor 10.10.10.10 remote-as 5511
 neighbor 10.10.10.10 update-source Loopback0
 neighbor 10.10.10.10 next-hop-self
 neighbor 10.10.10.10 soft-reconfiguration inbound
 neighbor 10.12.12.12 remote-as 5511
 neighbor 10.12.12.12 update-source Loopback0
 neighbor 10.12.12.12 next-hop-self
 neighbor 10.12.12.12 soft-reconfiguration inbound
 !
 ! eBGP -------------------------------
 neighbor 46.88.20.6 remote-as 23344                 
 neighbor 46.88.20.6 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 46.87.162.111 mask 255.255.255.255     ! Lo1
 network 46.87.162.0 mask 255.255.255.0
 network 46.88.20.0 mask 255.255.255.0
 network 46.88.22.0 mask 255.255.255.0
 network 46.89.36.0 mask 255.255.255.0
 network 46.89.38.0 mask 255.255.255.0
 network 46.92.2.0 mask 255.255.255.0
 no auto-summary
```

```txt
! Router 12 =================================================================
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 5511
 bgp router-id 10.12.12.12
 no synchronization
 bgp log-neighbor-changes
 !
 ! iBGP -------------------------------
 neighbor 10.10.10.10 remote-as 5511
 neighbor 10.10.10.10 update-source Loopback0
 neighbor 10.10.10.10 next-hop-self
 neighbor 10.10.10.10 soft-reconfiguration inbound
 neighbor 10.11.11.11 remote-as 5511
 neighbor 10.11.11.11 update-source Loopback0
 neighbor 10.11.11.11 next-hop-self
 neighbor 10.11.11.11 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 46.87.162.112 mask 255.255.255.255    ! Lo1
 network 46.87.162.1 mask 255.255.255.255      ! Server5
```

__AS 23344__ - Disney

```txt
! Router 13 =================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 158.23.228.0 255.255.255.0 Null0
! 
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 23344
 bgp router-id 10.13.13.13
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 46.88.20.5 remote-as 5511                 
 neighbor 46.88.20.5 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 158.23.228.113 mask 255.255.255.255    ! Lo1
 network 158.23.228.0 mask 255.255.255.0
 network 158.23.228.0 mask 255.255.255.252      ! avoid black-hole / access server6
 no auto-summary
```

__AS 701__ - Verizon

```txt
! Router 14 =================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 64.96.0.0 255.240.0.0 Null0
ip route 64.112.0.0 255.240.0.0 Null0
ip route 65.0.204.0 255.255.255.0 Null0
ip route 64.96.0.0 255.255.255.252 fastEthernet0/0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 701
 bgp router-id 10.14.14.14
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 64.112.0.10 remote-as 4637                 
 neighbor 64.112.0.10 soft-reconfiguration inbound
 neighbor 64.112.0.2 remote-as 1273                 
 neighbor 64.112.0.2 soft-reconfiguration inbound
 neighbor 64.112.0.6 remote-as 17390                 
 neighbor 64.112.0.6 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 64.96.0.114 mask 255.255.255.255    ! Lo1
 network 64.96.0.0 mask 255.240.0.0
 network 64.96.0.0 mask 255.255.255.252      ! avoid black-hole / access server3
 network 64.112.0.0 mask 255.240.0.0
 network 65.0.204.0 mask 255.255.255.0
 no auto-summary
```

__AS 4637__ - Telstra

```txt
! Router 15 =================================================================
!
! ===========================================================================
!                               STATIC ROUTES
! ===========================================================================
!
ip route 211.176.128.0 255.255.255.0 Null0
ip route 211.176.129.0 255.255.255.0 Null0
ip route 211.176.130.0 255.255.254.0 Null0
ip route 211.176.132.0 255.255.255.0 Null0
ip route 211.176.135.0 255.255.255.0 Null0
ip route 211.176.136.0 255.255.252.0 Null0
ip route 211.176.137.0 255.255.255.0 Null0
ip route 211.176.138.0 255.255.255.0 Null0
ip route 211.176.139.0 255.255.255.0 Null0
ip route 211.176.140.0 255.255.255.0 Null0
ip route 211.176.141.0 255.255.255.0 Null0
ip route 211.176.142.0 255.255.255.0 Null0
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 4637
 bgp router-id 10.15.15.15
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 64.112.0.9 remote-as 701                 
 neighbor 64.112.0.9 soft-reconfiguration inbound
 neighbor 48.73.240.13 remote-as 1273                 
 neighbor 48.73.240.13 soft-reconfiguration inbound
 neighbor 211.176.129.2 remote-as 5511                 
 neighbor 211.176.129.2 soft-reconfiguration inbound
 neighbor 211.176.129.6 remote-as 1                           
 neighbor 211.176.129.6 soft-reconfiguration inbound
 !
 ! Advertise Routes -------------------
 network 211.176.128.115 mask 255.255.255.255    ! Lo1
 network 211.176.128.0 mask 255.255.255.0
 network 211.176.128.0 mask 255.255.255.252      ! avoid black-hole / access server4
 network 211.176.129.0 mask 255.255.255.0
 network 211.176.130.0 mask 255.255.254.0
 network 211.176.132.0 mask 255.255.255.0
 network 211.176.135.0 mask 255.255.255.0
 network 211.176.136.0 mask 255.255.252.0
 network 211.176.137.0 mask 255.255.255.0
 network 211.176.138.0 mask 255.255.255.0
 network 211.176.139.0 mask 255.255.255.0
 network 211.176.140.0 mask 255.255.255.0
 network 211.176.141.0 mask 255.255.255.0
 network 211.176.142.0 mask 255.255.255.0
 no auto-summary
 ```
 
 __AS 20717__ - DE-CIX

```txt
! Router 16 =================================================================
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 20717
 bgp router-id 10.16.16.16
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 48.73.240.21 remote-as 1273                 
 neighbor 48.73.240.21 soft-reconfiguration inbound
 neighbor 46.88.20.1 remote-as 5511                 
 neighbor 46.88.20.1 soft-reconfiguration inbound
 ! Without Loopback 1 to advertise
 no auto-summary
```

__AS 1__ - Level 3 Parent

```txt
! BadGuy ====================================================================
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1
 bgp router-id 10.66.66.66
 no synchronization
 bgp log-neighbor-changes
 !
 ! eBGP -------------------------------
 neighbor 211.176.129.5 remote-as 4637                 
 neighbor 211.176.129.5 soft-reconfiguration inbound
 no auto-summary
```

### 2.3 - Test and Validation

#### BGP Status

__AS 1273__ - Vodafone

```txt
! Router 1 ==================================================================
!
R1#show ip bgp summary
BGP router identifier 10.1.1.1, local AS number 1273
BGP table version is 698, main routing table version 698
48 network entries using 6336 bytes of memory
109 path entries using 5668 bytes of memory
13/9 BGP path/bestpath attribute entries using 2184 bytes of memory
5 BGP rrinfo entries using 120 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 3 (at peak 4) using 96 bytes of memory
BGP using 14572 total bytes of memory
BGP activity 72/24 prefixes, 454/345 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.3.3.3        4       1273     318     261      698    0    0 03:47:15       47
10.4.4.4        4       1273     306     243      698    0    0 03:56:16       47
48.73.240.2     4      17390     282     268      698    0    0 03:56:32       11
```

```txt
! Router 2 ==================================================================
!
R2#show ip bgp summary
BGP router identifier 10.2.2.2, local AS number 1273
BGP table version is 802, main routing table version 802
48 network entries using 6336 bytes of memory
97 path entries using 5044 bytes of memory
9/8 BGP path/bestpath attribute entries using 1512 bytes of memory
5 BGP rrinfo entries using 120 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 1) using 32 bytes of memory
BGP using 13188 total bytes of memory
BGP activity 97/49 prefixes, 429/332 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.3.3.3        4       1273     344     265      802    0    0 03:49:31       46
10.4.4.4        4       1273     330     256      802    0    0 04:15:19       46
```

```txt
! Router 3 ==================================================================
!
R3#show ip bgp summary
BGP router identifier 10.3.3.3, local AS number 1273
BGP table version is 356, main routing table version 356
48 network entries using 6336 bytes of memory
88 path entries using 4576 bytes of memory
14/9 BGP path/bestpath attribute entries using 2352 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 3 (at peak 5) using 96 bytes of memory
BGP using 13552 total bytes of memory
BGP activity 86/38 prefixes, 314/226 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4       1273     238     281      356    0    0 03:50:48       10
10.2.2.2        4       1273     233     281      356    0    0 03:50:33        5
10.4.4.4        4       1273     276     280      356    0    0 03:51:03        4
10.5.5.5        4       1273     249     281      356    0    0 03:50:43        9
10.6.6.6        4       1273     265     281      356    0    0 03:51:02       30
48.73.240.6     4      17390     282     260      356    0    0 03:51:03       25
```

```txt
! Router 4 ==================================================================
!
R4#show ip bgp summary
BGP router identifier 10.4.4.4, local AS number 1273
BGP table version is 495, main routing table version 495
48 network entries using 6336 bytes of memory
69 path entries using 3588 bytes of memory
9/8 BGP path/bestpath attribute entries using 1512 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 2 (at peak 3) using 64 bytes of memory
BGP using 11644 total bytes of memory
BGP activity 96/48 prefixes, 376/307 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4       1273     272     348      495    0    0 04:01:11       10
10.2.2.2        4       1273     258     332      495    0    0 04:17:45        5
10.3.3.3        4       1273     343     339      495    0    0 03:52:26       11
10.5.5.5        4       1273     280     330      495    0    0 04:17:39        9
10.6.6.6        4       1273     291     332      495    0    0 04:17:54       30
```

```txt
! Router 5 ==================================================================
!
R5#sh ip bgp summary
BGP router identifier 10.5.5.5, local AS number 1273
BGP table version is 702, main routing table version 702
48 network entries using 6336 bytes of memory
113 path entries using 5876 bytes of memory
12/8 BGP path/bestpath attribute entries using 2016 bytes of memory
5 BGP rrinfo entries using 120 bytes of memory
9 BGP AS-PATH entries using 216 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 3 (at peak 4) using 96 bytes of memory
BGP using 14660 total bytes of memory
BGP activity 77/29 prefixes, 526/413 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.3.3.3        4       1273     347     302      702    0    0 03:52:56       42
10.4.4.4        4       1273     332     282      702    0    0 04:18:30       42
64.112.0.1      4        701     214     319      702    0    0 02:45:54       25
```

```txt
! Router 6 ==================================================================
!
R6#show ip bgp summary
BGP router identifier 10.6.6.6, local AS number 1273
BGP table version is 571, main routing table version 571
48 network entries using 6336 bytes of memory
129 path entries using 6708 bytes of memory
17/9 BGP path/bestpath attribute entries using 2856 bytes of memory
5 BGP rrinfo entries using 120 bytes of memory
13 BGP AS-PATH entries using 312 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 2 (at peak 4) using 64 bytes of memory
BGP using 16396 total bytes of memory
BGP activity 77/29 prefixes, 617/488 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.3.3.3        4       1273     348     311      571    0    0 03:54:04       21
10.4.4.4        4       1273     334     292      571    0    0 04:19:34       21
48.73.240.14    4       4637     216     335      571    0    0 02:36:39       31
48.73.240.18    4       5511     314     329      571    0    0 03:10:40       26
48.73.240.22    4      20717     185     313      571    0    0 04:20:15       26
```

__AS 17390__ - IBM

```txt
! Router 7 ==================================================================
!
R7#show ip bgp summary
BGP router identifier 10.7.7.7, local AS number 17390
BGP table version is 521, main routing table version 521
48 network entries using 6336 bytes of memory
130 path entries using 6760 bytes of memory
20/9 BGP path/bestpath attribute entries using 3360 bytes of memory
11 BGP AS-PATH entries using 280 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 2 (at peak 4) using 64 bytes of memory
BGP using 16800 total bytes of memory
BGP activity 68/20 prefixes, 552/422 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.8.8.8        4      17390     291     314      521    0    0 04:16:39       42
48.73.240.5     4       1273     308     337      521    0    0 04:02:32       42
64.112.0.5      4        701     215     342      521    0    0 02:55:02       42
130.41.46.10    4      64513     216     322      521    0    0 04:17:23        2
```

```txt
! Router 8 ==================================================================
!
R8#show ip bgp summary
BGP router identifier 10.8.8.8, local AS number 17390
BGP table version is 489, main routing table version 489
48 network entries using 6336 bytes of memory
93 path entries using 4836 bytes of memory
16/9 BGP path/bestpath attribute entries using 2688 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 2 (at peak 4) using 64 bytes of memory
BGP using 14116 total bytes of memory
BGP activity 74/26 prefixes, 378/285 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.7.7.7        4      17390     314     292      489    0    0 04:16:59       46
48.73.240.1     4       1273     301     315      489    0    0 04:11:54       42
130.41.46.6     4      64513     217     308      489    0    0 04:17:43        2
```

__AS 64513__ - Private (inside IBM)

```txt
! Router 9 ==================================================================
!
R9#show ip bgp summary
BGP router identifier 10.9.9.9, local AS number 64513
BGP table version is 358, main routing table version 358
48 network entries using 6336 bytes of memory
94 path entries using 4888 bytes of memory
10/7 BGP path/bestpath attribute entries using 1680 bytes of memory
7 BGP AS-PATH entries using 184 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 2) using 32 bytes of memory
BGP using 13120 total bytes of memory
BGP activity 62/14 prefixes, 204/110 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
130.41.46.5     4      17390     308     217      358    0    0 02:50:50       46
130.41.46.9     4      17390     323     217      358    0    0 02:50:50       46
```

__AS 5511__ - Orange

```txt
! Router 10 =================================================================
!
R10#show ip bgp summary
BGP router identifier 10.10.10.10, local AS number 5511
BGP table version is 224, main routing table version 224
48 network entries using 6336 bytes of memory
112 path entries using 5824 bytes of memory
19/9 BGP path/bestpath attribute entries using 3192 bytes of memory
15 BGP AS-PATH entries using 392 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 2 (at peak 4) using 64 bytes of memory
BGP using 15808 total bytes of memory
BGP activity 57/9 prefixes, 251/139 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.11.11.11     4       5511     211     239      224    0    0 03:07:51       10
10.12.12.12     4       5511     204     228      224    0    0 03:10:40        1
46.88.20.2      4      20717     133     222      224    0    0 03:20:09       22
48.73.240.17    4       1273     226     222      224    0    0 03:20:28       36
211.176.129.1   4       4637     151     231      224    0    0 02:46:48       36
```

```txt
! Router 11 =================================================================
!
R11#sh ip bgp summary
BGP router identifier 10.11.11.11, local AS number 5511
BGP table version is 176, main routing table version 176
48 network entries using 6336 bytes of memory
54 path entries using 2808 bytes of memory
9/8 BGP path/bestpath attribute entries using 1512 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 3 (at peak 4) using 96 bytes of memory
BGP using 10896 total bytes of memory
BGP activity 51/3 prefixes, 77/23 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4       5511     207     191      176    0    0 03:08:19       43
10.12.12.12     4       5511     192     191      176    0    0 03:08:44        1
46.88.20.6      4      23344     114     203      176    0    0 03:08:32        3
```

```txt
! Router 12 =================================================================
!
R12#sh ip bgp summary
BGP router identifier 10.12.12.12, local AS number 5511
BGP table version is 175, main routing table version 175
48 network entries using 6336 bytes of memory
54 path entries using 2808 bytes of memory
9/8 BGP path/bestpath attribute entries using 1512 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 1) using 32 bytes of memory
BGP using 10832 total bytes of memory
BGP activity 51/3 prefixes, 77/23 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4       5511     209     193      175    0    0 03:11:33       43
10.11.11.11     4       5511     192     192      175    0    0 03:09:09       10
```

__AS 23344__ - Disney

```txt
! Router 13 =================================================================
!
R13#show ip bgp summary
BGP router identifier 10.13.13.13, local AS number 23344
BGP table version is 542, main routing table version 542
48 network entries using 6336 bytes of memory
48 path entries using 2496 bytes of memory
9/8 BGP path/bestpath attribute entries using 1512 bytes of memory
6 BGP AS-PATH entries using 160 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 2) using 32 bytes of memory
BGP using 10536 total bytes of memory
BGP activity 180/132 prefixes, 241/193 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
46.88.20.5      4       5511     325     166      542    0    0 01:52:12       45
```

__AS 701__ - Verizon

```txt
! Router 14 =================================================================
!
R14#show ip bgp summary
BGP router identifier 10.14.14.14, local AS number 701
BGP table version is 94, main routing table version 94
48 network entries using 6336 bytes of memory
120 path entries using 6240 bytes of memory
21/9 BGP path/bestpath attribute entries using 3528 bytes of memory
17 BGP AS-PATH entries using 440 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 2) using 32 bytes of memory
BGP using 16576 total bytes of memory
BGP activity 65/17 prefixes, 196/76 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
64.112.0.2      4       1273     190     121       94    0    0 01:49:16       43
64.112.0.6      4      17390     192     121       94    0    0 01:49:17       29
64.112.0.10     4       4637     127     130       94    0    0 01:39:21       43
```

__AS 4637__ - Telstra

```txt
! Router 15 =================================================================
!
R15#show ip bgp summary
BGP router identifier 10.15.15.15, local AS number 4637
BGP table version is 49, main routing table version 49
48 network entries using 6336 bytes of memory
116 path entries using 6032 bytes of memory
22/9 BGP path/bestpath attribute entries using 3696 bytes of memory
18 BGP AS-PATH entries using 464 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 1) using 32 bytes of memory
BGP using 16560 total bytes of memory
BGP activity 50/2 prefixes, 118/2 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
48.73.240.13    4       1273     180     110       49    0    0 01:40:16       34
64.112.0.9      4        701     110     110       49    0    0 01:40:27       34
211.176.129.2   4       5511     181     110       49    0    0 01:40:38       34
211.176.129.6   4          1      93     110       49    0    0 01:39:23        0
```

__AS 20717__ - DE-CIX

```txt
! Router 16 =================================================================
!
R16#show ip bgp summary
BGP router identifier 10.16.16.16, local AS number 20717
BGP table version is 368, main routing table version 368
48 network entries using 6336 bytes of memory
96 path entries using 4992 bytes of memory
17/9 BGP path/bestpath attribute entries using 2856 bytes of memory
14 BGP AS-PATH entries using 352 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 2) using 32 bytes of memory
BGP using 14568 total bytes of memory
BGP activity 58/10 prefixes, 325/229 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
46.88.20.1      4       5511     327     208      368    0    0 01:51:13       48
48.73.240.21    4       1273     326     192      368    0    0 02:28:40       48
```

__AS 1__ - Level 3 Parent, LLC

```txt
! BadGuy =================================================================
!
BadGuy#show ip bgp summary
BGP router identifier 10.66.66.66, local AS number 1
BGP table version is 318, main routing table version 318
48 network entries using 6336 bytes of memory
48 path entries using 2496 bytes of memory
8/7 BGP path/bestpath attribute entries using 1344 bytes of memory
7 BGP AS-PATH entries using 184 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 10360 total bytes of memory
BGP activity 67/19 prefixes, 138/90 paths, scan interval 60 secs

Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
211.176.129.5   4       4637     217     152      318    0    0 01:32:06       48
```

### Prefixes received and sent from or to a determined neighbour

Example from Router 3 received and sent routes from / to Router 6

```txt
R3#show ip bgp neighbors 10.6.6.6 received-routes
BGP table version is 356, local router ID is 10.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i46.87.162.0/24   10.6.6.6                 0    100      0 5511 i
*>i46.87.162.110/32 10.6.6.6                 0    100      0 5511 i
*>i46.87.162.111/32 10.6.6.6                 0    100      0 5511 i
*>i46.87.162.112/32 10.6.6.6                 0    100      0 5511 i
*>i46.88.20.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.88.22.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.36.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.38.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.92.2.0/24     10.6.6.6                 0    100      0 5511 i
* i48.73.239.0/24   10.6.6.6                 0    100      0 i
*>i48.73.239.66/32  10.6.6.6                 0    100      0 i
* i48.73.240.0/22   10.6.6.6                 0    100      0 i
* i63.25.64.0/18    10.6.6.6                 0    100      0 i
*>i158.23.228.0/30  10.6.6.6                 0    100      0 5511 23344 i
*>i158.23.228.0/24  10.6.6.6                 0    100      0 5511 23344 i
*>i158.23.228.113/32
                    10.6.6.6                 0    100      0 5511 23344 i
*>i211.176.128.0/30 10.6.6.6                 0    100      0 4637 i
*>i211.176.128.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.128.115/32
                    10.6.6.6                 0    100      0 4637 i
*>i211.176.129.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.130.0/23 10.6.6.6                 0    100      0 4637 i
*>i211.176.132.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.135.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.136.0/22 10.6.6.6                 0    100      0 4637 i
*>i211.176.137.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.138.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.139.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.140.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.141.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.142.0    10.6.6.6                 0    100      0 4637 i

Total number of prefixes 30

! ----------------------------------------------------------------------

R3#show ip bgp neighbors 10.6.6.6 advertised-routes
BGP table version is 356, local router ID is 10.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i46.87.162.0/24   10.6.6.6                 0    100      0 5511 i
*>i46.87.162.110/32 10.6.6.6                 0    100      0 5511 i
*>i46.87.162.111/32 10.6.6.6                 0    100      0 5511 i
*>i46.87.162.112/32 10.6.6.6                 0    100      0 5511 i
*>i46.88.20.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.88.22.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.36.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.38.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.92.2.0/24     10.6.6.6                 0    100      0 5511 i
*>i48.73.239.0/30   10.2.2.2                 0    100      0 i
*> 48.73.239.0/24   0.0.0.0                  0         32768 i
*>i48.73.239.11/32  10.1.1.1                 0    100      0 i
*>i48.73.239.22/32  10.2.2.2                 0    100      0 i
*> 48.73.239.33/32  0.0.0.0                  0         32768 i
*>i48.73.239.44/32  10.4.4.4                 0    100      0 i
*>i48.73.239.55/32  10.5.5.5                 0    100      0 i
*>i48.73.239.66/32  10.6.6.6                 0    100      0 i
*> 48.73.240.0/22   0.0.0.0                  0         32768 i
*> 48.73.240.4/30   0.0.0.0                  0         32768 i
*> 63.25.64.0/18    0.0.0.0                  0         32768 i
*>i64.96.0.0/30     10.5.5.5                 0    100      0 701 i
*>i64.96.0.0/12     10.5.5.5                 0    100      0 701 i
*>i64.96.0.114/32   10.5.5.5                 0    100      0 701 i
*>i64.112.0.0/12    10.5.5.5                 0    100      0 701 i
*>i65.0.204.0/24    10.5.5.5                 0    100      0 701 i
*> 130.41.46.0/30   48.73.240.6                            0 17390 i
*> 130.41.46.0/24   48.73.240.6              0             0 17390 i
*> 130.41.46.77/32  48.73.240.6              0             0 17390 i
*> 130.41.46.88/32  48.73.240.6                            0 17390 i
*> 130.41.47.0/24   48.73.240.6                            0 17390 64513 i
*> 130.41.47.99/32  48.73.240.6                            0 17390 64513 i
*>i158.23.228.0/30  10.6.6.6                 0    100      0 5511 23344 i
*>i158.23.228.0/24  10.6.6.6                 0    100      0 5511 23344 i
*>i158.23.228.113/32
                    10.6.6.6                 0    100      0 5511 23344 i
*>i211.176.128.0/30 10.6.6.6                 0    100      0 4637 i
*>i211.176.128.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.128.115/32
                    10.6.6.6                 0    100      0 4637 i
*>i211.176.129.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.130.0/23 10.6.6.6                 0    100      0 4637 i
*>i211.176.132.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.135.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.136.0/22 10.6.6.6                 0    100      0 4637 i
*>i211.176.137.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.138.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.139.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.140.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.141.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.142.0    10.6.6.6                 0    100      0 4637 i

Total number of prefixes 48
```

#### Prefixes received and sent from or to a determined neighbour, notably the prefixes installed on the Loc-RIB BGP table

Example from Router 9

```txt
R9#show ip bgp
BGP table version is 358, local router ID is 10.9.9.9
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  46.87.162.0/24   130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  46.87.162.110/32 130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*> 46.87.162.111/32 130.41.46.5                            0 17390 1273 5511 i
*                   130.41.46.9                            0 17390 1273 5511 i
*  46.87.162.112/32 130.41.46.5                            0 17390 1273 5511 i
*>                  130.41.46.9                            0 17390 1273 5511 i
*  46.88.20.0/24    130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  46.88.22.0/24    130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  46.89.36.0/24    130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  46.89.38.0/24    130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  46.92.2.0/24     130.41.46.9                            0 17390 1273 5511 i
*>                  130.41.46.5                            0 17390 1273 5511 i
*  48.73.239.0/30   130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.239.0/24   130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.239.11/32  130.41.46.5                            0 17390 1273 i
*>                  130.41.46.9                            0 17390 1273 i
*  48.73.239.22/32  130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.239.33/32  130.41.46.5                            0 17390 1273 i
*>                  130.41.46.9                            0 17390 1273 i
*  48.73.239.44/32  130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.239.55/32  130.41.46.5                            0 17390 1273 i
*>                  130.41.46.9                            0 17390 1273 i
*  48.73.239.66/32  130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.240.0/22   130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  48.73.240.4/30   130.41.46.5                            0 17390 1273 i
*>                  130.41.46.9                            0 17390 1273 i
*  63.25.64.0/18    130.41.46.9                            0 17390 1273 i
*>                  130.41.46.5                            0 17390 1273 i
*  64.96.0.0/30     130.41.46.5                            0 17390 701 i
*>                  130.41.46.9                            0 17390 701 i
*  64.96.0.0/12     130.41.46.5                            0 17390 701 i
*>                  130.41.46.9                            0 17390 701 i
*  64.96.0.114/32   130.41.46.5                            0 17390 701 i
*>                  130.41.46.9                            0 17390 701 i
*  64.112.0.0/12    130.41.46.5                            0 17390 701 i
*>                  130.41.46.9                            0 17390 701 i
*  65.0.204.0/24    130.41.46.5                            0 17390 701 i
*>                  130.41.46.9                            0 17390 701 i
*  130.41.46.0/30   130.41.46.9                            0 17390 i
*>                  130.41.46.5              0             0 17390 i
*  130.41.46.0/24   130.41.46.9              0             0 17390 i
*>                  130.41.46.5              0             0 17390 i
*  130.41.46.77/32  130.41.46.5                            0 17390 i
*>                  130.41.46.9              0             0 17390 i
*  130.41.46.88/32  130.41.46.9                            0 17390 i
*>                  130.41.46.5              0             0 17390 i
*> 130.41.47.0/24   0.0.0.0                  0         32768 i
*> 130.41.47.99/32  0.0.0.0                  0         32768 i
*> 158.23.228.0/30  130.41.46.5                            0 17390 1273 5511 23344 i
*                   130.41.46.9                            0 17390 1273 5511 23344 i
*> 158.23.228.0/24  130.41.46.5                            0 17390 1273 5511 23344 i
*                   130.41.46.9                            0 17390 1273 5511 23344 i
*> 158.23.228.113/32
                    130.41.46.5                            0 17390 1273 5511 23344 i
*                   130.41.46.9                            0 17390 1273 5511 23344 i
*  211.176.128.0/30 130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.128.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.128.115/32
                    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.129.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.130.0/23 130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.132.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.135.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.136.0/22 130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.137.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.138.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.139.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.140.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.141.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
*  211.176.142.0    130.41.46.5                            0 17390 1273 4637 i
*>                  130.41.46.9                            0 17390 701 4637 i
```

####  TCLSH script on the appendix TCLSH (Tool Control Language) Shell for testing

All routers passed the TCLSH ping tests with success, example from Router 12

```txt
R12#tclsh
R12(tcl)#foreach address {
+>48.73.239.11
+>48.73.239.22
+>48.73.239.33
+>48.73.239.44
+>48.73.239.55
+>48.73.239.66
+>130.41.46.77
+>130.41.46.88
+>130.41.47.99
+>46.87.162.110
+>46.87.162.111
+>46.87.162.112
+>158.23.228.113
+>64.96.0.114
+>211.176.128.115
+>130.41.46.1
+>48.73.239.1
+>64.96.0.1
+>211.176.128.1
+>46.87.162.1
+>158.23.228.1
+>} {ping $address source lo1}

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.11, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 60/81/132 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.22, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 40/49/60 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.33, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 40/53/64 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.44, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/32/48 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.55, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 48/67/92 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.66, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/24/36 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 130.41.46.77, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 52/63/76 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 130.41.46.88, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 60/73/88 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 130.41.47.99, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 52/80/104 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 46.87.162.110, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/13/20 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 46.87.162.111, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/12/16 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 46.87.162.112, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 158.23.228.113, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/42/84 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 64.96.0.114, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 52/94/152 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 211.176.128.115, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/26/52 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 130.41.46.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 72/93/116 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 48.73.239.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 56/72/96 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 64.96.0.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 80/93/100 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 211.176.128.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 32/44/64 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 46.87.162.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/18/44 ms
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 158.23.228.1, timeout is 2 seconds:
Packet sent with a source address of 46.87.162.112
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/39/84 ms
```

### 2.4 - Practical Questions

#### 2.4.1 - How is the BGP next hop reachability solved inside an AS?

Inside an Autonomous System (AS), BGP next-hop reachability is resolved using the Interior Gateway Protocol (IGP), such as OSPF.
When an eBGP-learned route is advertised via iBGP, the next-hop IP address remains unchanged by default (it is the IP of the external peer).
To ensure that all routers within the AS can reach this next-hop IP, we use the next-hop-self command on iBGP sessions.
This command changes the next-hop attribute to the router’s own IP (usually the loopback address), which is already reachable via the IGP.

#### 2.4.2 - Why is it a good practice to use the loopback IP address in the iBGP sessions? 

Using a loopback interface for iBGP sessions offers:

- __Stability__: Loopback interfaces are always up unless administratively shut down, unlike physical interfaces.
-  __Resilience__: If a physical link fails, the router can still reach the loopback via alternate paths.
-  __Scalability__: Simplifies configuration in route reflector and peer-group setups.
-  __Consistency__: Ensures the BGP session is not tied to a specific physical interface.

#### 2.4.3 - Create and present a detailed table with all the connectivity tests performed using the TCLSH procedure, as previously outlined

__TODO__

| AS                 | Source Router(s) | Destination IP | Result |
| ------------------ | ---------------- | -------------- | ------ |
| AS 1273 - Vodafone | R1, R2, R3, R4, R5, R6 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success | 
|  |  |  |   |
| AS 17390 - IBM | R7, R8 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success | 
|  |  |  |   |
| AS 64513 - Private (inside IBM) | R9 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success | 
|  |  |  |   |
| AS 5511 - Orange | R10, R11, R12 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success | 
|  |  |  |   |
| AS 23344 - Disney | R13 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success | 
|  |  |  |   |
| AS 701 - Verizon | R14 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success |
|  |  |  |   |
| AS 4637 - Telstra | R15 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success |
|  |  |  |   |
| AS 20717 - DE-CIX | R16 | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success |
|  |  |  |   |
| AS 1 - Level 3 Parent, LCC | BadGuy | 48.73.239.11 (R1 Lo1)| Success |
|  |  | 48.73.239.22 (R2 Lo1) | Success |
|  |  | 48.73.239.33 (R3 Lo1)| Success |
|  |  | 48.73.239.44 (R4 Lo1)	 | Success | 
|  |  | 48.73.239.55 (R5 Lo1)	 | Success | 
|  |  | 48.73.239.66 (R6 Lo1)	 | Success |
|  |  | 130.41.46.77 (R7 Lo1)	 | Success |
|  |  | 130.41.46.88 (R8 Lo1)	 | Success | 
|  |  | 130.41.47.99 (R9 Lo1)	 | Success |
|  |  | 46.87.162.110 (R10 Lo1) | Success |
|  |  | 46.87.162.111 (R11 Lo1) | Success | 
|  |  | 46.87.162.112 (R12 Lo1) | Success | 
|  |  | 158.23.228.113 (R13 Lo1)| Success |
|  |  | 64.96.0.114 (R14 Lo1) | Success | 
|  |  | 211.176.128.115 (R15 Lo1) | Success |
|  |  | 130.41.46.1 (Server1) | Success | 
|  |  | 48.73.239.1 (Server2) | Success |
|  |  | 64.96.0.1 (Server3)	| Success | 
|  |  | 211.176.128.1 (Server4) | Success | 
|  |  | 46.87.162.1 (Server5) | Success |
|  |  | 158.23.228.1 (Server6) | Success |

#### 2.4.4 - In the following Local-RIB table output example, how many routes were installed for the destination 46.87.162.0/24? Justify your response explaining the decision process in BGP

```txt
     Network         Next Hop       Metric LocPrf Weight Path
* 46.87.162.0/24     130.41.46.5                       0 17390 1273 20717 5511 i
*>                   130.41.46.9                       0 17390 1273 20717 5511 i
```

Only one route is installed in the routing table — the one marked with *>.
The * indicates a valid route, and > indicates the best path.
BGP selects only one best path per prefix based on its decision process (Weight → Local Preference → AS Path → etc.).
Here, both paths have the same attributes, so the tie-breaker is likely the lowest router ID or lowest neighbor IP.
Only the best path is installed in the routing table and advertised to other BGP peers.

#### 2.4.5 - What would have happened in case you didn’t configure the next-hop-self on the iBGP peering definitions? What reasons explain why BGP doesn’t set the next-hop-self as a default setting?

If `next-hop-self` is not configured:

- The next-hop IP of eBGP-learned routes remains the external peer’s IP.
-  Internal routers (iBGP peers) may not have a route to that external IP in their routing table.
-  This leads to unreachable next-hop and the BGP route is not used.

__Why it’s not default__:

BGP assumes that the next-hop IP is reachable via the IGP. In some designs (e.g., MPLS VPNs, or when using static routes), the next-hop may be intentionally preserved.
Making ``next-hop-self` the default could break such designs, so it is left as an explicit configuration choice.

#### 2.4.6 - How are the route prefixes propagated inside the AS when you have route reflectors configured?

In a route reflector (RR) setup:

- The RR reflects iBGP-learned routes to its clients and other RRs.
- Clients only peer with the RR, not with each other (reduces full-mesh requirement).
- The RR preserves BGP attributes like NEXT_HOP, AS_PATH, and LOCAL_PREF.
- Routes learned from an eBGP peer are reflected to all iBGP clients and other RRs.
- Routes learned from a client are reflected to all other clients and non-client peers (including eBGP).

This allows scalable propagation of prefixes across the AS without a full iBGP mesh.

#### 2.4.7 - Simulate and explain using Wireshark the BGP messages associated to each BGP state (see: [article](https://www.ciscopress.com/articles/article.asp?p=2756480&seqNum=4) )

__TODO__ error in mac OSX

BGP uses a state machine with the following states and associated messages:

| BGP State	| Message Type	 | Description |
| -------- | ------------- | ------------ |
| Idle	| – | Initial state; no resources allocated. | 
| Connect | TCP SYN	| Attempts to establish TCP connection (port 179). | 
| Active	| TCP SYN | Actively tries to connect if passive fails. | 
| OpenSent	| OPEN | TCP established; sends BGP OPEN message. | 
| OpenConfirm	| KEEPALIVE	| Waits for KEEPALIVE; if received, moves to Established. | 
| Established	UPDATE	Session is up; routes are exchanged via UPDATE messages. | 

Wireshark Capture Notes:

- OPEN: Contains AS number, BGP version, hold time, and BGP identifier.
- KEEPALIVE: Maintains session alive.
- UPDATE: Advertises or withdraws routes.
- NOTIFICATION: Used for error reporting or session termination.

In Wireshark, filter with tcp.port == 179 to observe BGP messages and correlate them with BGP state transitions.

<div style="page-break-after: always"></div>

## 3 - Phase 3 - Routing Policies Implementation

### 3.1 - Objectives

In this phase, we have as objectives:

- Aggregate IP prefixes when possible and advertise only the aggregate route on eBGP sessions
- We should not have advertised prefixes longer than /24 in the internet routing table
- The internet peering’s should accept a maximum of 50 prefixes
- We should not have private ASs in the middle from an AS path attribute
- The AS 23344 has only one peering to the internet and want to receive only the default route from the eBGP peering

### 3.2 - Implementation

In the ASes with one peer router only and that advertise routes, we used the `aggregate-address` command.

```txt
! AS 64513 (private to IBM) - Router 9 =====================================
router bgp 64513
 bgp router-id 10.9.9.9
 ! ...
 network 130.41.47.0 mask 255.255.255.0
 aggregate-address 130.41.47.0 255.255.255.0 summary-only

! AS 23344 Disney - Router 13 ==============================================
router bgp 23344
 bgp router-id 10.13.13.13
 ! ...
 network 158.23.228.0 mask 255.255.255.0
 aggregate-address 158.23.228.0 255.255.255.0 summary-only
 
! AS 701 Verizon - Router 14 ===============================================
router bgp 701
 bgp router-id 10.14.14.14
 ! ...
 network 64.96.0.0 mask 255.240.0.0
 network 64.112.0.0 mask 255.240.0.0
 network 65.0.204.0 mask 255.255.255.0
 aggregate-address 64.96.0.0 255.255.255.0 summary-only
 aggregate-address 64.112.0.0 255.255.255.0 summary-only
 aggregate-address 65.0.204.0 255.255.255.0 summary-only
 
! AS 4637 Telstra - Router 15 ===============================================
router bgp 4637
 bgp router-id 10.15.15.15
 ! ...
 network 211.176.128.0 mask 255.255.252.0
 network 211.176.132.0 mask 255.255.255.0 
 network 211.176.135.0 mask 255.255.255.0 
 network 211.176.136.0 mask 255.255.252.0 
 network 211.176.140.0 mask 255.255.254.0 
 network 211.176.141.0 mask 255.255.255.0
 aggregate-address 211.176.128.0 255.255.252.0 summary-only
 aggregate-address 211.176.132.0 255.255.255.0 summary-only
 aggregate-address 211.176.135.0 255.255.255.0 summary-only
 aggregate-address 211.176.136.0 255.255.252.0 summary-only
 aggregate-address 211.176.140.0 255.255.254.0 summary-only
 aggregate-address 211.176.141.0 255.255.255.0 summary-only
```

On the ASes with more than one router, we created an access list restricting the prefixes to advertise to the eBGP peers

```txt
! AS 1273 Vodafone =========================================================
!
! All routers  -------------------------------------------------------------
ip prefix-list PREFIX_LIST_ADV_SUMM seq 5 permit 0.0.0.0/0 le 24
!
! Router 1 -----------------------------------------------------------------
router bgp 1273
 bgp router-id 10.1.1.1
 ! ...
 neighbor 48.73.240.2 remote-as 17390
 neighbor 48.73.240.2 soft-reconfiguration inbound
 neighbor 48.73.240.2 prefix-list PREFIX_LIST_ADV_SUMM out
!
! Router 3 -----------------------------------------------------------------
router bgp 1273
 bgp router-id 10.3.3.3
 ! ...
 neighbor 48.73.240.6 remote-as 17390
 neighbor 48.73.240.6 soft-reconfiguration inbound
 neighbor 48.73.240.6 prefix-list PREFIX_LIST_ADV_SUMM out
!
! Router 5 -----------------------------------------------------------------
router bgp 1273
 bgp router-id 10.5.5.5
 ! ...
 neighbor 64.112.0.1 remote-as 701
 neighbor 64.112.0.1 soft-reconfiguration-inbound
 neighbor 64.112.0.1 prefix-list PREFIX_LIST_ADV_SUMM out
!
! Router 6 -----------------------------------------------------------------
router bgp 1273
 bgp router-id 10.6.6.6
 ! ...
 neighbor 48.73.240.14 remote-as 4637
 neighbor 48.73.240.14 soft-reconfiguration inbound
 neighbor 48.73.240.14 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 48.73.240.22 remote-as 20717
 neighbor 48.73.240.22 soft-reconfiguration inbound
 neighbor 48.73.240.22 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 48.73.240.18 remote-as 5511
 neighbor 48.73.240.18 soft-reconfiguration inbound
 neighbor 48.73.240.18 prefix-list PREFIX_LIST_ADV_SUMM out
```

```txt
! AS 17390 IBM =============================================================
!
! Router 7 -----------------------------------------------------------------
router bgp 17390
 bgp router-id 10.7.7.7
 ! ...
 neighbor 64.112.0.5 remote-as 701
 neighbor 64.112.0.5 soft-reconfiguration inbound
 neighbor 64.112.0.5 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 64.112.0.5 remove-private-as
 neighbor 130.41.46.10 remote-as 64513
 neighbor 130.41.46.10 soft-reconfiguration inbound
 neighbor 130.41.46.10 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 48.73.240.5 remote-as 1273
 neighbor 48.73.240.5 soft-reconfiguration inbound
 neighbor 48.73.240.5 prefix-list PREFIX_LIST_ADV_SUMM out
!
! Router 8 -----------------------------------------------------------------
router bgp 17390
 bgp router-id 10.8.8.8
 ! ...
 neighbor 130.41.46.6 remote-as 64513
 neighbor 130.41.46.6 soft-reconfiguration inbound
 neighbor 130.41.46.6 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 48.73.240.1 remote-as 1273
 neighbor 48.73.240.1 soft-reconfiguration inbound
 neighbor 48.73.240.1 prefix-list PREFIX_LIST_ADV_SUMM out
```

```txt
! AS 5511 Orange ===========================================================
!
! Router 10 ----------------------------------------------------------------
router bgp 5511
 bgp router-id 10.10.10.10
 ! ...
 neighbor 46.88.20.2 remote-as 20717
 neighbor 46.88.20.2 soft-reconfiguration inbound
 neighbor 46.88.20.2 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 48.73.240.17 remote-as 1273
 neighbor 48.73.240.17 soft-reconfiguration inbound
 neighbor 48.73.240.17 prefix-list PREFIX_LIST_ADV_SUMM out
 neighbor 211.176.129.1 remote-as 4637
 neighbor 211.176.129.1 soft-reconfiguration inbound
 neighbor 211.176.129.1 prefix-list PREFIX_LIST_ADV_SUMM out
!
! Router 11 ----------------------------------------------------------------
router bgp 5511
 bgp router-id 10.11.11.11
 ! ...
 neighbor 46.88.20.6 remote-as 23344              
 neighbor 46.88.20.6 soft-reconfiguration inbound
 neighbor 46.88.20.6 prefix-list PREFIX_LIST_ADV_SUMM out
```

To limit the internet peering’s to a maximum of 50 prefixes, in all the BGP's `neighbour`, we use the command `neighbor [peer IP address] maximum-prefix 50`

To avoid the private AS (AS 64513) to be announced outside to the internet, in the routers R7 and R8, we used the command: `neighbor [peer IP address] remove-private-as`. This prevented the advertising of the network announced by the router R9 (AS 64513) outside the AS 17390.

The AS 23344 has only one peering to the internet and want to receive only the default route from the eBGP peering. To accomplish that, in the router R11 that belongs to the only AS (AS 5511 - Orange) that connects to R13 of AS 23344 - Disney, we used the command `neighbor 46.88.20.6 default-originate`

### 3.3 - Test and Validation

Inspection of the __AS_PATH__ attribute in each router

To accomplish this, we use the commands:
 - `show ip bgp 211.176.129.0`
 - `show ip bgp`

 The private AS 64515 is not announced to the internet, only to the AS 17390 IBM (routers R7 and R8). As an output example, we annexed the output of the router R3:

```txt
R3#show ip bgp 211.176.129.0
BGP routing table entry for 211.176.128.0/22, version 32
Paths: (1 available, best #1, table Default-IP-Routing-Table)
Flag: 0x820
  Advertised to update-groups:
        1    2    4
  4637, (aggregated by 4637 10.15.15.15), (Received from a RR-client), (received & used)
    10.6.6.6 (metric 3) from 10.6.6.6 (10.6.6.6)
      Origin IGP, metric 0, localpref 100, valid, internal, atomic-aggregate, best
      
R3#show ip bgp
BGP table version is 32, local router ID is 10.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i46.87.162.0/24   10.6.6.6                 0    100      0 5511 i
*>i46.88.20.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.88.22.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.36.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.89.38.0/24    10.6.6.6                 0    100      0 5511 i
*>i46.92.2.0/24     10.6.6.6                 0    100      0 5511 i
*>i48.73.239.0/30   10.2.2.2                 0    100      0 i
* i48.73.239.0/24   10.2.2.2                 0    100      0 i
* i                 10.4.4.4                 0    100      0 i
* i                 10.5.5.5                 0    100      0 i
* i                 10.1.1.1                 0    100      0 i
* i                 10.6.6.6                 0    100      0 i
*>                  0.0.0.0                  0         32768 i
*>i48.73.239.11/32  10.1.1.1                 0    100      0 i
*>i48.73.239.22/32  10.2.2.2                 0    100      0 i
*> 48.73.239.33/32  0.0.0.0                  0         32768 i
*>i48.73.239.44/32  10.4.4.4                 0    100      0 i
*>i48.73.239.55/32  10.5.5.5                 0    100      0 i
*>i48.73.239.66/32  10.6.6.6                 0    100      0 i
* i48.73.240.0/22   10.2.2.2                 0    100      0 i
* i                 10.4.4.4                 0    100      0 i
* i                 10.5.5.5                 0    100      0 i
* i                 10.1.1.1                 0    100      0 i
* i                 10.6.6.6                 0    100      0 i
*>                  0.0.0.0                  0         32768 i
*> 48.73.240.4/30   0.0.0.0                  0         32768 i
* i63.25.64.0/18    10.2.2.2                 0    100      0 i
* i                 10.4.4.4                 0    100      0 i
* i                 10.5.5.5                 0    100      0 i
* i                 10.1.1.1                 0    100      0 i
* i                 10.6.6.6                 0    100      0 i
*>                  0.0.0.0                  0         32768 i
*  64.96.0.0/24     48.73.240.6                            0 17390 701 i
*>i                 10.5.5.5                 0    100      0 701 i
*  64.96.0.0/12     48.73.240.6                            0 17390 701 i
*>i                 10.5.5.5                 0    100      0 701 i
*  64.112.0.0/12    48.73.240.6                            0 17390 701 i
*>i                 10.5.5.5                 0    100      0 701 i
*  65.0.204.0/24    48.73.240.6                            0 17390 701 i
*>i                 10.5.5.5                 0    100      0 701 i
*> 130.41.46.0/24   48.73.240.6              0             0 17390 i
* i                 10.1.1.1                 0    100      0 17390 i
*> 130.41.47.0/24   48.73.240.6                            0 17390 i
*>i158.23.228.0/24  10.6.6.6                 0    100      0 5511 23344 i
*>i211.176.128.0/22 10.6.6.6                 0    100      0 4637 i
*>i211.176.132.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.135.0    10.6.6.6                 0    100      0 4637 i
*>i211.176.136.0/22 10.6.6.6                 0    100      0 4637 i
*>i211.176.140.0/23 10.6.6.6                 0    100      0 4637 i
```

The output of router 8, with the private AS advertised:

```txt
R8# Show ip bgp
...

* i130.41.47.0/24   10.7.7.7                 0    100      0 64513 i
*>                  130.41.46.6              0             0 64513 i

...
```

### 3.4 - Practical Questions

#### 3.4.1 - After applying your prefix-list or route-map, compare the output of `show ip bgp` and `show ip bgp neighbors <ip> advertised-routes`. Explain the differences you observe referring to the __Adj-RIB-In__, __Adj-RIB-Out__, and __Loc-RIB__ tables

In our lab, we applied the following outbound prefix filter to eBGP peers to ensure only prefixes with a mask of /24 or less are advertised:

```txt
ip prefix-list PREFIX_LIST_ADV_SUMM seq 5 permit 0.0.0.0/0 le 24
```

This prefix list permits any prefix with a subnet mask length ≤ 24 and implicitly denies longer prefixes (e.g., /32 loopback addresses).

Comparison of Command Outputs:

- `show ip bgp` (Loc-RIB Table)
This command displays all BGP routes that are valid and installed in the local BGP table after inbound policies have been applied. It includes:
    - All prefixes learned from all peers (both internal and external).
    - Prefixes with masks longer than /24 (e.g., `/32` Loopback1 addresses).
    - All candidate routes for best-path selection.
- `show ip bgp neighbors <ip> advertised-routes` (Adj-RIB-Out Table)
This command shows only the routes that have passed outbound policy filters and will be advertised to the specified neighbor. After applying PREFIX_LIST_ADV_SUMM, this table will:
    - Exclude all `/32` prefixes (Loopback1 addresses).
    - Include only prefixes with masks `/24 or shorter (e.g., server subnets like `46.87.162.0/24`).
    - Reflect the effect of any summary-only aggregate commands.

Example:

In `show ip bgp`, we see:

```text
48.73.239.11/32
46.87.162.0/24
```

In `show ip bgp neighbors 48.73.240.6 advertised-routes`, we see:

```text
46.87.162.0/24
```

But not `48.73.239.11/32`.

__BGP Table Relationship Summary__:

- __Adj-RIB-In__: Routes received from neighbor → __before__ inbound policy.
- __Loc-RIB__: After inbound policy + best-path selection → __used for local routing__.
- __Adj-RIB-Out__: After outbound policy → what is __advertised to neighbor__.

So, our prefix list `PREFIX_LIST_ADV_SUMM` acts as an __outbound filter__ that removes longer prefixes (like `/32` loopbacks) from the __Adj-RIB-Out__ table, ensuring only summarized or appropriately sized prefixes are advertised to external peers. This aligns with BGP best practices to reduce routing table size and improve scalability

#### 3.4.2 - Why is the control from the number of prefixes advertised to the internet a good practice?

Controlling the number of prefixes advertised is important for:

- __Scalability__: Prevents the global BGP table from growing excessively, which would increase memory and CPU usage on routers.
- __Stability__: Reduces the risk of route flapping and BGP convergence issues.
- __Security__: Limits the impact of route leaks or hijacks.
- __Policy compliance__: Many ISPs enforce prefix limits in peering agreements to ensure efficient routing.

In this lab, limiting prefixes to a maximum of 50 per peer helps maintain a manageable and stable routing environment

> __Route flapping__ refers to the rapid and repeated change in the reachability of a network prefix in the routing table. This occurs when a BGP route is alternately advertised and withdrawn in quick succession due to:
>
> - Interface instability (physical or logical link going up/down)
> - Configuration changes
> - Network congestion or errors
> - Unstable peering sessions
> - Router software/hardware issues

#### 3.4.3 - What is the private AS number range in BGP? Describe some scenarios where using private ASNs can be useful.

Private ASN Ranges:

| Format | Private Range  |
| ------ | -------------- |
| 2-byte | 64512 to 65534 |
| 4-byte | 4200000000 to 4294967294 |

Use cases for private ASNs:

- __Internal network segmentation__: Within a larger AS, private ASNs can be used for testing or staging environments without consuming public AS numbers.
- __Mergers and migrations__: During network integration, private ASNs can be used temporarily to avoid conflicts.
- __Customer networks__: In some BGP designs, customers may use a private ASN for their internal routing, which is removed before advertisement to the global Internet (as done in this lab with AS 17390).

#### 3.4.4 - Assume that you successfully configured prefix filtering. Based on the __BGP path selection rules__, explain why R12 selects a specific path for the prefix 65.0.204.0/24. Use the following output as reference [link](https://www.cisco.com/en/US/tech/tk365/technologies_tech_note09186a0080094431.shtml):

```txt
R12# sh ip bgp 65.0.204.0
BGP routing table entry for 65.0.204.0/24, version 18717
Paths: (2 available, best #2, table Default-IP-Routing-Table)
   Advertised to update-groups:
       2
  4637 701, (received & used)
    10.10.10.10 (metric 2) from 10.10.10.10 (10.10.10.10)
     Origin IGP, metric 0, localpref 100, valid, internal
  1273 701, (received & used)
    48.73.240.17 from 48.73.240.17 (10.6.6.6)
     Origin IGP, localpref 100, valid, external, best  
```

R12 selects the path via __1273 701__ as best because:

1. Both paths have the same __Local Preference (100)__.
2. The path via __1273 701__ is __external__ (eBGP), while the other is __internal__ (iBGP). BGP prefers __eBGP over iBGP__ when other attributes are equal.
3. The __AS_PATH__ length is the same (2 AS hops).
4. No difference in __MED__, __Origin__, or other attributes to influence the decision.

#### 3.4.5 - Imagine that one of your prefixes was not selected as the best path in your lab. Based on the __BGP decision process__, propose a configuration change (e.g., adjusting local preference, MED, or weight) that would alter the selection. Justify your answer by showing the relevant command(s) and predicting the impact on the routing table.

Supposing that prefix `46.88.20.0/24` is not selected as best because it comes via an AS with longer __AS_PATH__. To influence selection, we could increase the __Local Preference__ for that path.

Configuration change on R12 (inbound from desired neighbor):

```txt
route-map PREFER_PATH permit 10
 match ip address prefix-list PREFIX_LIST
 set local-preference 200
!
router bgp 5511
 neighbor 48.73.240.18 route-map PREFER_PATH in
```

Increasing __Local Preference__ to 200 makes this path more preferred than others with default __Local Preference__ (100). This change influences the first step of BGP decision process, ensuring this path is selected as best.

<div style="page-break-after: always"></div>

## 4 - Phase 4 - Influence the Internet Routing

### 4.1 - Objectives

In this phase the objectives are:

- Use BGP attributes to ensure __AS20717__ is the preferred path for traffic originated in __AS1273__ and __AS511__
- Enforce this preference symmetrically, guaranteeing consistent selection in both directions.
- Apply the rule along the full path (e.g., AS17390 <-> AS1273 <-> IXP1 <-> AS551) for routing coherence

### 4.2 - Implementation

To ensure that __AA20717__ is the preferred path for traffic originated in __AS1273__ with destination or passing through __AS511__, or traffic originated in __AS511__ with destination or passing through __AS1273__, we first created an AS-Path  Access List that matches the desired route.

```txt
Router6 - AS1273 ===========================================================
! AS-PATH Access List for traffic engineering
! Matches routes that have passed through both AS20717 and AS5511
! Syntax: _20717_5511$ = ends with AS5511 and has AS20717 somewhere before it
ip as-path access-list 1 permit _20717_5511$

Router10 - AS5511 ==========================================================
! AS-PATH Access List for traffic engineering
! Matches routes that have passed through AS20171 then AS1273
! Syntax: _20171_1273$ = ends with AS1273 and has AS20171 somewhere before it
ip as-path access-list 1 permit _20171_1273$
``` 

Then, we created route map with `weight 300` matching the previously configured AS-Path ACL, in both routers (R6 - AS1273 - Vodafone, and R10 - AS5511 - Orange

```txt
Router6 - AS1273 ===========================================================
! Route-map for traffic engineering - preferring routes via AS20717+AS5511
! Sets WEIGHT=300 for routes that come via AS20717 then AS5511
! WEIGHT is Cisco proprietary (local to router, highest value wins)
route-map RM_PREF_LOCAL_AS5511 permit 10
 match as-path 1                  ! Match routes with AS-PATH _20717_5511$
 set weight 300                   ! Set high weight to prefer these routes

Router10 - AS5511 ==========================================================
! Route-map for traffic engineering - preferring routes via AS20171+AS1273
! Sets WEIGHT=300 for routes that come via AS20171 then AS1273
! WEIGHT is Cisco proprietary (local to router, highest value wins)
! This influences Orange's preference for paths through DE-CIX then Vodafone
route-map RM_PREF_LOCAL_AS1273 permit 10
 match as-path 1                  ! Match routes with AS-PATH _20171_1273$
 set weight 300                   ! Set high weight to prefer these routes
```

At least, in the direct links connecting the transit __AS20717__ - DE-CIX, in the both routers we apply the Route Map

```txt
Router6 - AS1273 ===========================================================
router bgp 1273
! ...
 ! eBGP Peer 2: AS20717 (DE-CIX)
 neighbor 48.73.240.22 remote-as 20717
 ! ...
 ! Apply route-map for traffic engineering (prefer routes via AS20717+AS5511)
 neighbor 48.73.240.22 route-map RM_PREF_LOCAL_AS5511 in
 ! ...

Router10 - AS5511 ==========================================================
router bgp 5511
 ! ...
 ! eBGP Peer 1: AS20717 (DE-CIX)
 neighbor 46.88.20.2 remote-as 20717
 ! ...
 ! Apply route-map for traffic engineering (prefer routes via AS20171+AS1273)
 neighbor 46.88.20.2 route-map RM_PREF_LOCAL_AS1273 in
```

### 4.3 - Test and Validation

__TODO__

### 4.4 - Practical Questions

#### 4.4.1 - Describe in your report the policy options you used to implement the routing policies (include all the details of the configurations (e.g., prefix-list, route-map, etc)

Described and explained in the previous point 4.2 - Implementation

#### 4.4.2 - Provide command output screenshots that demonstrate the successful application of the configured policies

__TODO__

#### 4.4.3 - Discuss other alternative to achieve the same results and comment on their relative pros and cons, compared to your implementation

Alternative Approaches to Achieve the Same Result:

**1. Local Preference (LOCAL_PREF)**

```cisco
! Instead of WEIGHT (Cisco proprietary, local to router)
route-map RM_PREF_LOCAL_AS5511 permit 10
  match as-path 1
  set local-preference 200          ! Higher LOCAL_PREF wins (default is 100)
```

**Pros:**
- Standard BGP attribute (RFC 4271) - interoperable across vendors
- Propagated within the AS to all iBGP peers
- More granular control (can be set on per-neighbor basis)
- Well-understood and widely deployed

**Cons:**
- Affects all routers in the AS (global effect within the AS)
- Requires careful planning to avoid unintended consequences
- Cannot override LOCAL_PREF from other iBGP speakers within same AS

**Comparison to our WEIGHT implementation:**
- WEIGHT is router-local, LOCAL_PREF is AS-wide
- WEIGHT has higher precedence than LOCAL_PREF in Cisco's decision process
- Our WEIGHT approach is simpler for single-router policy but doesn't scale to multi-router AS

**2. AS_PATH Prepending**

```cisco
! Make other paths less desirable by artificially lengthening AS_PATH
route-map RM_PREPEND_OUT permit 10
  set as-path prepend 1273 1273 1273  ! Add our AS 3 times
```

Applied to outbound advertisements to other transit providers (not DE-CIX).

**Pros:**
- Simple to implement and understand
- Effective for making your AS less preferred by others
- Works well for outbound traffic engineering
- No special configuration needed on neighbor's side

**Cons:**
- Only affects inbound traffic to your AS
- Can be filtered by neighbors with AS_PATH length limits
- Less precise control compared to WEIGHT/LOCAL_PREF
- Doesn't help with our specific requirement (preferring certain transit paths)

**3. MED (Multi-Exit Discriminator)**

```cisco
! Lower MED is preferred (default is 0)
route-map RM_SET_MED permit 10
  match as-path 1
  set metric 50                     ! Lower MED for preferred paths
```

**Pros:**
- Standard BGP attribute
- Useful when you have multiple connections to same AS
- Can influence neighboring AS's routing decisions

**Cons:**
- Only comparable between routes from the same neighboring AS
- Often ignored by many ISPs for security/policy reasons
- Not transitive (only sent to direct neighbors)
- Less effective for our cross-AS path preference scenario

**4. Community-Based Traffic Engineering**

```cisco
! Tag routes with communities and apply policies based on them
route-map RM_TAG_COMMUNITIES permit 10
  match as-path 1
  set community 1273:100 20717:200  ! Tag with custom communities

! Neighbor configuration to send communities
neighbor 48.73.240.22 send-community
neighbor 48.73.240.22 route-map RM_TAG_COMMUNITIES out
```

**Pros:**
- Highly scalable and flexible
- Enables coordinated policies across multiple ASes
- Can implement sophisticated routing policies
- Well-supported by major ISPs

**Cons:**
- Requires agreement and coordination between ASes
- More complex to implement and debug
- Dependent on neighbor honoring your communities
- Not all ISPs accept or act on customer communities

**5. BGP Add-Path (RFC 7911)**

```cisco
! Enable multiple paths for same prefix
router bgp 1273
  bgp additional-paths select best 3
  neighbor 48.73.240.22 additional-paths send receive
```

**Pros:**
- Maintains multiple paths simultaneously
- Enables faster failover
- More granular path selection capabilities
- Future-proof approach

**Cons:**
- Requires support on both ends
- Increased memory and CPU usage
- More complex configuration
- Not yet universally deployed

<div style="page-break-after: always"></div>

## 5 - Phase 5 - Security Practices

### 5.1 Objectives

In this last phase, the objectives are:

- Activate the MD5 authentication on all the eBGP peering’s on the AS701. This a good practice for all the peers nevertheless for the lab proposes will be ok to test on this AS only
- Filter the `Bogon` prefixes on the peer routers from AS 1273. The BadGuy router is advertising `Bogons` (see: [BGP Best Practices for ISPs](https://conference.apnic.net/22/docs/tut-routing-pres-bgp-bcp.pdf)
- Implement Remote Triggered Black Hole (RTBH) filtering - a popular and effective technique for the mitigation of denial-of-service (DoS) attack on AS1273 coming from the prefix `63.96.0.115`. [Remotely Triggered Black Hole Filtering - Destination Based and Source Based](https://www.cisco.com/c/dam/en_us/about/security/intelligence/blackhole.pdf)

### 5.2 - Implementation

#### Enable MD5 Authentication on eBGP Peerings (AS701)

In all the routers (R5, R7, R14 and R15) we used the command `service password-encryption`, to prevent the visualisation of the password  when running the command `show running-configuration`

On router R14 from AS 701 - Verizon

```txt
router bgp 701
 bgp router-id 10.14.14.14
 ! ...
 neighbor 64.112.0.10 remote-as 4637
 neighbor 64.112.0.10 password BGP@p455w0rd_701-4637             
 ! ...
 neighbor 64.112.0.2 remote-as 1273
 neighbor 64.112.0.2 password BGP@p455w0rd_701-1273               
 ! ...
 neighbor 64.112.0.6 remote-as 17390
 neighbor 64.112.0.6 password BGP@p455w0rd_701-17390
 ! ...
```

On Router R5 from AS 1273 - Vodafone

```txt
router bgp 1273
 bgp router-id 10.5.5.5
 ! ...
 neighbor 64.112.0.1 remote-as 701
 neighbor 64.112.0.1 password BGP@p455w0rd_701-1273
 ! ...
```

On router R7 from AS 17390 IBM

```txt
router bgp 17390
 bgp router-id 10.7.7.7
 ! ...
 neighbor 64.112.0.5 remote-as 701
 neighbor 64.112.0.5 password BGP@p455w0rd_701-17390 
 ! ...
```

On router R15 from AS 4637 Telstra

```txt
router bgp 4637
 bgp router-id 10.15.15.15
 !...
 neighbor 64.112.0.9 remote-as 701
 neighbor 64.112.0.9 password BGP@p455w0rd_701-4637 
 ! ...
```

#### Filter Bogon Prefixes from BadGuy Router

In all the routers, we defined a __BOGON-FILTER__ with all the IP addresses that are not permitted in the internet

```txt
ip prefix-list BOGON-FILTER seq 5 deny 0.0.0.0/8 le 32        ! Software (current local network)
ip prefix-list BOGON-FILTER seq 10 deny 10.0.0.0/8 le 32      ! Private
ip prefix-list BOGON-FILTER seq 15 deny 100.64.0.0/10 le 32   ! Shared Address Space - CG-NAT
ip prefix-list BOGON-FILTER seq 20 deny 127.0.0.0/8 le 32     ! Loopback addresses to Localhost
ip prefix-list BOGON-FILTER seq 25 deny 169.254.0.0/16 le 32  ! APIPA - Automatic Private IP Address
ip prefix-list BOGON-FILTER seq 30 deny 172.16.0.0/12 le 32   ! Private
ip prefix-list BOGON-FILTER seq 35 deny 192.0.0.0/24 le 32    ! IETF Protocol Assignments
ip prefix-list BOGON-FILTER seq 40 deny 192.0.2.0/24 le 32    ! Documentation - TEST-NET-1
ip prefix-list BOGON-FILTER seq 45 deny 192.88.99.0/24 le 32  ! Reserved
ip prefix-list BOGON-FILTER seq 50 deny 192.168.0.0/16 le 32  ! Private
ip prefix-list BOGON-FILTER seq 55 deny 198.18.0.0/15 le 32   ! Benchmark testing
ip prefix-list BOGON-FILTER seq 60 deny 198.51.100.0/24 le 32 ! Documentation - TEST-NET-2
ip prefix-list BOGON-FILTER seq 65 deny 203.0.113.0/24 le 32  ! Documentation - TEST-NET-3
ip prefix-list BOGON-FILTER seq 70 deny 224.0.0.0/4 le 32     ! Multicast
ip prefix-list BOGON-FILTER seq 75 deny 233.252.0.0/24 le 32  ! Documentation - MCAST-TEST-NET
ip prefix-list BOGON-FILTER seq 80 deny 240.0.0.0/4 le 32     ! Reserved for future use
ip prefix-list BOGON-FILTER seq 90 permit 0.0.0.0/0 le 32     ! All address space
```

Then, in all the eBGP `neighbor` command we apply the filter to the input:

```txt
neighbor [neighbor ip address] prefix-list BOGON-FILTER in
```

In the router __BadGuy__, to simulate the advertising of `Bogons`, we created the `bogon` static routes pointing to `Null0` and in the BGP section we advertise them with the `redistribute static` command. This command allow the injection of all the static routes of the routing table in the BGP routing process

```txt
! ============================================================================
!                                BOGON PREFIXES
! ============================================================================
!
ip route 0.0.0.0 255.0.0.0 Null0           ! Software (current local network)
ip route 10.0.0.0 255.0.0.0 Null0          ! Private
ip route 100.64.0.0 255.192.0.0 Null0      ! Shared Address Space - CG-NAT
ip route 127.0.0.0 255.0.0.0 Null0         ! Loopback addresses to Localhost
ip route 169.254.0.0 255.255.0.0 Null0     ! APIPA - Automatic Private IP Address
ip route 172.16.0.0 255.240.0.0 Null0      ! Private
ip route 192.0.0.0 255.255.255.0 Null0     ! IETF Protocol Assignments
ip route 192.0.2.0 255.255.255.0 Null0     ! Documentation - TEST-NET-1
ip route 192.88.99.0 255.255.255.0 Null0   ! Reserved
ip route 192.168.0.0 255.255.0.0 Null0     ! Private
ip route 198.18.0.0 255.254.0.0 Null0      ! Benchmark testing
ip route 198.51.100.0 255.255.255.0 Null0  ! Documentation - TEST-NET-2
ip route 203.0.113.0 255.255.255.0 Null0   ! Documentation - TEST-NET-3
ip route 224.0.0.0 240.0.0.0 Null0         ! Multicast
ip route 233.252.0.0 255.255.255.0 Null0   ! Documentation - MCAST-TEST-NET
ip route 240.0.0.0 240.0.0.0 Null0         ! Reserved for future use
!
! ===========================================================================
!                                   BGP
! ===========================================================================
!
router bgp 1
 bgp router-id 10.66.66.66
 !...
 redistribute static
 ```

#### Apply Remote Triggered Black Hole (RTBH) filtering to mitigate a DoS attack within AS1273

To simulate an attack from the Router __R14__ to the __AS 1273__ - Vodafone, we created the interface `Loopback 2` with the IP address `63.96.0.115 255.255.255.255`.

Inside the __AS 1273__ - Vodafone, we configured the Router __R4__ as a `trigger router` to filter the attacking prefix

```txt
! ===========================================================================
! Blackhole Routing Configuration (BGP Traffic Scrubbing / DDoS Mitigation)
! ===========================================================================

! ---------------------------------------------------------------------------
! 1. Static Route to Null0 (Blackhole Route)
!    - Creates a discard route for a specific host (64.96.0.115/32)
!    - Traffic matching this route will be silently dropped by the router
!    - TAG 66: Used for route tracking or identification (can be matched in route-maps)
! ---------------------------------------------------------------------------
ip route 64.96.0.115 255.255.255.255 Null0 tag 66

! ---------------------------------------------------------------------------
! 2. Prefix List for Route Matching
!    - Defines the specific prefix (64.96.0.115/32) to be matched in route-maps
!    - Used to selectively apply BGP policies to this specific host route
! ---------------------------------------------------------------------------
ip prefix-list BH seq 5 permit 64.96.0.115/32

! ---------------------------------------------------------------------------
! 3. Route Map for BGP Announcement Manipulation
!    - Applied when advertising this prefix to BGP peers
!    - "black-hole-trigger": Descriptive name indicating DDoS mitigation trigger
!    - Sequence 10: First (and likely only) policy statement
! ---------------------------------------------------------------------------
route-map black-hole-trigger permit 10
  ! Match the specific blackhole prefix defined above
  match ip address prefix-list BH
  
  ! Set BGP LOCAL_PREFERENCE to 200 (high priority within local AS)
  ! This makes the blackhole route preferred over regular paths
  set local-preference 200
  
  ! Set BGP ORIGIN attribute to IGP (indicates internal origin)
  set origin igp
  
  ! Add NO-EXPORT community (well-known community 0xFFFFFF01)
  ! Prevents this route from being advertised outside the local AS
  set community no-export
  
  ! Modify NEXT_HOP to a specific scrubbing center IP (192.0.2.1)
  ! This redirects traffic to a DDoS mitigation/scrubbing appliance
  set ip next-hop 192.0.2.1

! -----------------------------------------------------------------------------
! 4. Static Route to Scrubbing Center
!    - Routes traffic destined to the scrubbing center (192.0.2.1) to Null0
!    - This is a LOOP PREVENTION mechanism:
!       1. BGP announces 64.96.0.115/32 with next-hop 192.0.2.1
!       2. Traffic to victim (64.96.0.115) goes to scrubbing center (192.0.2.1)
!       3. This route ensures traffic to scrubbing center itself doesn't loop
!       4. Actual scrubbing center reachability via other means (not shown)
! -----------------------------------------------------------------------------
ip route 192.0.2.1 255.255.255.255 Null0

! IN BGP SECTION
router bgp 1273
 redistribute static        ! Redistributes static routes (includes 64.96.0.115/32)
 ! ...
 !---------------------------------------------------------------------------
 ! CRITICAL: Apply blackhole route-map to outgoing advertisements
 !    - Applies the 'black-hole-trigger' route-map to modify BGP attributes
 !    - This ensures blackhole routes get special treatment when advertised
 ! --------------------------------------------------------------------------
 neighbor iBGP route-map black-hole-trigger out
 ! ...
``` 

### 5.3 - Test and Validation

__TODO__

### 5.4 - Practical Questions

#### 5.4.1 - How does MD5 authentication improve the security of BGP sessions? What happens if it is not enabled?

**1. Protection Against Session Hijacking**

MD5 authentication prevents unauthorized routers from establishing BGP sessions with your router by requiring a shared secret key.

**2. Defense Against Replay Attacks**

The MD5 hash includes a sequence number/timestamp, making it resistant to replay attacks where an attacker captures and resends valid BGP packets.

**3. Prevention of Rogue Peer Establishment**

Without authentication, any router claiming to be from a legitimate AS could form a BGP session and:

- Inject malicious routes
- Hijack prefixes
- Cause routing instability

**4. Integrity Verification**

Each BGP packet is checksummed with the secret key, ensuring the packet hasn't been tampered with in transit.

**5. Basic Peer Identity Verification**

While not strong cryptographic authentication, MD5 provides a basic level of identity verification for BGP peers.

##### **What Happens Without MD5 Authentication**

**Critical Vulnerabilities Exposed:**

**1. Route Hijacking Attacks**

```cisco
# Example of what an attacker could do:
# 1. Spoof BGP OPEN message from legitimate peer
# 2. Establish unauthorized BGP session
# 3. Announce hijacked prefixes:
network 8.8.8.0/24    # Hijack Google DNS
network 203.0.113.0/24 # Announce bogon space
```

**Real-World Example**: In 2018, attackers hijacked Amazon's Route 53 DNS prefixes, redirecting traffic to malicious servers. [link](https://blog.cloudflare.com/bgp-leaks-and-crypto-currencies/)

**2. BGP Session Reset Attacks**

Without authentication, attackers can send:
- **TCP RST packets** to tear down legitimate BGP sessions
- **BGP NOTIFICATION messages** to cause session resets
- **Malformed BGP packets** to crash BGP processes

**3. Route Poisoning/Instability**

Attackers can inject:
- **Malformed attributes** (corrupted AS_PATH, communities)
- **Excessive updates** causing CPU exhaustion
- **Withdrawals for critical prefixes** causing blackholes

**4. Man-in-the-Middle Attacks**

Legitimate Path:   AS1273 <--> [Internet] <--> AS5511
Compromised Path:  AS1273 <--> [Attacker] <--> AS5511

Attacker can:

1. Intercept and modify BGP updates
2. Learn internal network topology
3. Perform traffic analysis
4. Selectively drop or modify routes


**5. Prefix De-aggregation Attacks**

Legitimate announcement:   192.0.2.0/24
Attacker announcement:     192.0.2.0/25
                          192.0.2.128/25
                          
More specific routes win, allowing traffic hijacking.

**Impact of Missing Authentication - Case Study**

**The YouTube Hijack Incident (2008)**

WHAT HAPPENED:

- Pakistan Telecom (AS17557) announced YouTube's prefix (208.65.153.0/24)
- No authentication between Pakistan Telecom and PCCW (AS3491)
- Route propagated globally
- YouTube inaccessible worldwide for ~2 hours

ROOT CAUSE:

1. No BGP session authentication
2. No route filtering
3. No RPKI/ROV validation
4. No maximum prefix limits

**Financial Impact:**
- **Direct costs**: $1.5M+ in lost ad revenue (YouTube)
- **Indirect costs**: Brand damage, customer trust erosion
- **Operational costs**: Emergency response, mitigation efforts

[Wikipedia page about Google Outages](https://en.wikipedia.org/wiki/Google_services_outages)

## **Conclusion**

**Without MD5 authentication, BGP sessions are vulnerable to:**
- ✓ Session hijacking
- ✓ Route poisoning  
- ✓ Traffic interception
- ✓ Denial of service
- ✓ Prefix hijacking

**MD5 authentication is a fundamental, minimum-security requirement** that should be deployed on ALL eBGP sessions. While MD5 itself has cryptographic weaknesses, it's vastly better than no authentication at all and provides a critical first layer of defense in BGP security.

The ideal approach is **defense-in-depth**: combine MD5/TCP-AO authentication with prefix filtering, AS_PATH validation, RPKI/ROV, and operational monitoring to create a robust BGP security posture.

#### 5.4.2 - Which Bogon ranges did you filter in your lab, and why must they not appear in the global routing table?

##### Filtered Ranges:

__Private/Internal Space__:

- `10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16` - Corporate/Home networks
- `100.64.0.0/10` - Carrier-Grade NAT (ISP internal use)

__Special/Reserved__:

- `0.0.0.0/8` - "This" network
- `127.0.0.0/8` - Loopback (localhost)
- `169.254.0.0/16` - Link-local auto-configuration

__Documentation/Testing__:

- `192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24` - Test networks
- `233.252.0.0/24` - Multicast testing

##### Reasons why must they not appear in the global routing table

1. __Security Threats__:
 - Spoofing attacks - Fake sources enable DDoS
 - Route hijacking - Redirect internal traffic
 - Resource exhaustion - Fill routing tables with junk

2. Breaks Networks:
 - VPNs fail when private routes leak
 - NAT stops working
 - Cloud services conflict

3. Violates Standards:
 - RFC 1918: Private addresses must stay local
 - BCP 38: Required ingress filtering
 - MANRS: Industry security requirement

4. Internet Stability:
 - Pollutes global BGP table (~900K routes)
 - Causes routing confusion globally
 - Wastes ISP resources

#### 5.4.3 - Explain how RTBH was implemented in AS1273. Include a diagram showing how the trigger router propagates the blackhole route

__TODO__

#### 5.4.4 - Describe the role of uRPF in validating traffic source addresses and preventing spoofing

**uRPF (Unicast Reverse Path Forwarding) - Anti-Spoofing Defense**

Core Function:

- Validates that incoming packets arrive on the interface that would be used to send packets BACK to the source address.**

**How uRPF Works:**

Three Operating Modes:

**1. Strict Mode (`ip verify unicast source reachable-via rx`)**

```cisco
interface GigabitEthernet0/1
 ip verify unicast source reachable-via rx
```

**Logic:** "Packet must arrive on THE EXACT interface my routing table would use to reach the source."
- **Check:** Does RIB/FIB have a route to source? Is the best path via this interface?
- **Drop if:** Source isn't routable OR best return path ≠ incoming interface
- **Best for:** Single-homed networks, edge interfaces

**2. Loose Mode (`ip verify unicast source reachable-via any`)**

```cisco
interface GigabitEthernet0/1
 ip verify unicast source reachable-via any
```

**Logic:** "Packet source must be routable SOMEWHERE in my table."
- **Check:** Does any route exist to source? (Any interface)
- **Drop if:** Source isn't routable at all
- **Best for:** Multi-homed networks, core interfaces

**3. VRF Mode (Per-VRF checking)**

```cisco
interface GigabitEthernet0/1
 ip verify unicast vrf CUSTOMER-A source reachable-via rx
```

**Logic:** Same as strict/loose but within specific VRF routing table.

**Key Benefits:**

**1. Stops DDoS Reflection/Amplification**

```text
Common DDoS: Spoof victim's IP → DNS/NTP servers → Amplified reply floods victim
With uRPF: Spoofed packets dropped at first hop
```

**2. Prevents IP Address Hijacking**

- Can't impersonate other networks
- Can't use bogon addresses
- Can't spoof internal resources

**3. Simple but Effective**

- Single command implementation
- Hardware-accelerated on most ASICs
- Minimal performance impact

**4. Compliance Ready**

- Meets PCI-DSS, NIST, ISO 27001 requirements
- Implements BCP 38/RFC 2827
- MANRS Action 4 recommendation

uRPF acts as a "return address checker" for network traffic, dropping any packets that arrive on the "wrong door" based on your routing table, making source address spoofing virtually impossible at the network edge.

#### 5.4.5 - What impact would the attack from 64.96.0.115 have on AS1273 if RTBH was not deployed?

__TODO__

<div style="page-break-after: always"></div>

## Conclusion

__TODO__

<div style="page-break-after: always"></div>

## Appendices

### IP Addressing for Lab Topology

#### Router 1

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.1.1.1/32        |                   |
| Lo1       |                    | 48.73.239.11/32   |
| g1/0      | 10.1.2.1/30        |                   |
| g2/0      | 10.1.3.1/30        |                   |
| g4/0      |                    | 48.73.240.1/30    |

#### Router 2

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.2.2.2/32        |                   |
| Lo1       |                    | 48.73.239.22/32   |
| f0/0      |                    | 48.73.239.2/30    |
| g1/0      | 10.1.2.2/30        |                   |
| g2/0      | 10.2.4.1/30        |                   |

#### Router 3

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.3.3.3/32        |                   |
| Lo1       |                    | 48.73.239.33/32   |
| g1/0      | 10.3.4.1/30        |                   |
| g2/0      | 10.1.3.2/30        |                   |
| g3/0      | 10.3.5.1/30        |                   |
| g4/0      |                    | 48.73.240.5/30    |

#### Router 4

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.4.4.4/32        |                   |
| Lo1       |                    | 48.73.239.44/32   |
| g1/0      | 10.3.4.2/30        |                   |
| g2/0      | 10.2.4.2/30        |                   |
| g3/0      | 10.4.6.1/30        |                   |


#### Router 5

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.5.5.5/32        |                   |
| Lo1       |                    | 48.73.239.55/32   |
| g1/0      |                    | 64.112.0.2/30     |
| g3/0      | 10.3.5.2/30        |                   |

#### Router 6

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.6.6.6/32        |                   |
| Lo1       |                    | 48.73.239.66/32   |
| f0/0      |                    | 48.73.240.17/30   |
| g1/0      |                    | 48.73.240.13/30   |
| g2/0      |                    | 48.73.240.21/30   |
| g3/0      | 10.4.6.2/30        |                   |

#### Router 7

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.7.7.7/32        |                   |
| Lo1       |                    | 130.41.46.77/32   |
| f0/0      |                    | 130.41.46.9/30    |
| g1/0      | 10.7.8.1/30        |                   |
| g4/0      |                    | 48.73.240.6/30    |
| g5/0      |                    | 64.112.0.6/30     |

#### Router 8

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.8.8.8/32        |                   |
| Lo1       |                    | 130.41.46.88/32   |
| f0/0      |                    | 130.41.46.5/30    |
| g1/0      | 10.7.8.2/30        |                   |
| g4/0      |                    | 48.73.240.2/30    |
| g5/0      |                    | 130.41.46.2/30    |

#### Router 9

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.9.9.9/32        |                   |
| Lo1       |                    | 130.41.47.99/32   |
| f0/0      |                    | 130.41.46.10/30   |
| f0/1      |                    | 130.41.46.6/30    |

#### Router 10

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.10.10.10/32     |                   |
| Lo1       |                    | 46.87.162.110/32  |
| g1/0      | 10.10.11.1/30      |                   |
| g2/0      |                    | 211.176.129.2/30  |
| g3/0      | 10.10.12.1/30      |                   |
| g4/0      |                    | 46.88.20.1/30     |

#### Router 11

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.11.11.11/32     |                   |
| Lo1       |                    | 46.87.162.111/32  |
| f0/0      |                    | 46.88.20.5/30     |
| g1/0      | 10.10.11.2/30      |                   |
| g2/0      | 10.11.12.1/30      |                   |

#### Router 12

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.12.12.12/32     |                   |
| Lo1       |                    | 46.87.162.112/32  |
| f0/0      |                    | 46.87.162.2/30    |
| g2/0      | 10.11.12.2/30      |                   |
| g3/0      | 10.10.12.2/30      |                   |
| g5/0      |                    | 48.73.240.18/30   |

#### Router 13

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.13.13.13/32     |                   |
| Lo1       |                    | 158.23.228.113/32 |
| f0/0      |                    | 46.88.20.6/30     |
| g1/0      |                    | 158.23.228.2/30   |

#### Router 14

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.14.14.14/32     |                   |
| Lo1       |                    | 64.96.0.114/32    |
| f0/0      |                    | 64.96.0.2/30      |
| g1/0      |                    | 64.112.0.1/30     |
| g3/0      |                    | 64.112.0.9/30     |
| g5/0      |                    | 64.112.0.5/30     |

#### Router 15

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.15.15.15/32     |                   |
| Lo1       |                    | 211.176.128.115/32 |
| f0/0      |                    | 211.176.128.2/30  |
| g1/0      |                    | 48.73.240.14/30   |
| g2/0      |                    | 211.176.129.1/30  |
| g3/0      |                    | 64.112.0.10/30    |
| g4/0      |                    | 211.176.129.5/30  |

#### Router 16

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.16.16.16/32     |                   |
| Lo1       |                    |                   |
| g2/0      |                    | 48.73.240.22/30   |
| g4/0      |                    | 48.88.20.2/30     |

#### Badguy

| Interface | Private IP Address | Public IP Address |
| --------- | ------------------ | ----------------- |
| Lo0       | 10.66.66.66/32     |                   |
| g4/0      |                    | 211.176.129.6/30  |

#### Servers

| Name    | Interface | Public IP Address |
| ------- | --------- | ----------------- |
| Server1 | e0        | 130.41.46.1/30    |
| Server2 | e0        | 48.73.239.1/30    |
| Server3 | e0        | 64.96.0.1/30      |
| Server4 | e0        | 211.176.128.1/30  |
| Server5 | e0        | 46.87.162.1/30    |
| Server6 | e0        | 158.23.228.1/30   |

<div style="page-break-after: always"></div>

### TCLSH (Tool Control Language) Shell for Testing

```txt
R9# tclsh
R9(tcl)#
foreach address {
48.73.239.11
48.73.239.22
48.73.239.33
48.73.239.44
48.73.239.55
48.73.239.66
130.41.46.77
130.41.46.88
130.41.47.99
46.87.162.110
46.87.162.111
46.87.162.112
158.23.228.113
64.96.0.114
211.176.128.115
130.41.46.1
48.73.239.1
64.96.0.1
211.176.128.1
46.87.162.1
158.23.228.1
} {ping $address source lo1}
```