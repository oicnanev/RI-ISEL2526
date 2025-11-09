# Lab Project N.º 2 - BGP ROUTING CONFIGURATIONS

> REDES DE INTERNET (RI) 2025-2026

---

## Group 1 - Class 51D

### Professor Luís Mata

> luis.mata@isel.pt

- 45824 Nuno Venâncio
- 49420 André Carrilho
- 51454 Hugo Leal

![Lab Project 2 topology](./assets/01.png)

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
R1(config)#Int Loopback 0
R1(config-if)#ip address 10.1.1.1 255.255.255.255

R1(config-if)#int Loopback 1
R1(config-if)#ip address 48.73.239.11 255.255.255.255

R1(config-if)#int g1/0
R1(config-if)#ip address 10.1.2.1 255.255.255.252
R1(config-if)#no shut

R1(config-if)#int g2/0
R1(config-if)#ip address 10.1.3.1 255.255.255.252
R1(config-if)#no shut

R1(config-if)#int g4/0
R1(config-if)#ip address 48.73.240.1 255.255.255.252
R1(config-if)#no shut

R1(config-if)#do wr
Building configuration...
[OK]
```

Server example:

```txt
VPCS> set pcname Server1

Server1> ip 130.41.46.1/30 130.41.46.2
Checking for duplicate address...
Server1 : 130.41.46.1 255.255.255.252 gateway 130.41.46.2

Server1> save
Saving startup configuration to startup.vpc
.  done
```

#### OSPF Configuration

__AS 1273__ - Vodafone

```txt
! All Routers -------------------------------------------------------------
R#(config) router ospf 1

! ROUTER 1 ----------------------------------------------------------------
R1(config-router)# router-id 10.1.1.1
R1(config-router)# network 10.1.2.0 0.0.0.3 area 0      ! g1/0 -> R2
R1(config-router)# network 10.1.3.0 0.0.0.3 area 0      ! g2/0 -> R3
R1(config-router)# network 10.1.1.1 0.0.0.0 area 0      ! Lo0

! ROUTER 2 ----------------------------------------------------------------
R2(config-router)# router-id 10.2.2.2
R2(config-router)# network 10.1.2.0 0.0.0.3 area 0      ! g1/0 -> R1
R2(config-router)# network 10.2.4.0 0.0.0.3 area 0      ! g2/0 -> R4
R2(config-router)# network 10.2.2.2 0.0.0.0 area 0      ! Lo0

! ROUTER 3 ----------------------------------------------------------------
R3(config-router)# router-id 10.3.3.3
R3(config-router)# network 10.3.4.0 0.0.0.3 area 0      ! g1/0 -> R4
R3(config-router)# network 10.1.3.0 0.0.0.3 area 0      ! g2/0 -> R1
R3(config-router)# network 10.3.5.0 0.0.0.3 area 0      ! g3/0 -> R5
R3(config-router)# network 10.3.3.3 0.0.0.0 area 0      ! Lo0

! ROUTER 4 ----------------------------------------------------------------
R4(config-router)#router-id 10.4.4.4
R4(config-router)#network 10.3.4.2 0.0.0.3 area 0       ! g1/0 -> R3
R4(config-router)#network 10.2.4.2 0.0.0.3 area 0       ! g2/0 -> R2
R4(config-router)#network 10.4.6.1 0.0.0.3 area 0       ! g3/0 -> R6
R4(config-router)#network 10.4.4.4 0.0.0.0 area 0       ! lo0

! ROUTER 5 ----------------------------------------------------------------
R5(config-router)#router-id 10.5.5.5
R5(config-router)#network 10.3.5.0 0.0.0.3 area 0       ! g3/0 -> R3
R5(config-router)#network 10.5.5.5 0.0.0.0 area 0       !lo0

! ROUTER 6 ----------------------------------------------------------------
R6(config-router)#router-id 10.6.6.6
R6(config-router)#network 10.4.6.0 0.0.0.3 area 0       ! g3/0 -> R4
R6(config-router)#network 10.6.6.6 0.0.0.0 area 0       ! lo0
```

__AS 17390__ - IBM

```txt
! ROUTER 7 ----------------------------------------------------------------
R7(config-if)#router ospf 1
R7(config-router)#router-id 10.7.7.7
R7(config-router)#network 10.7.8.0 0.0.0.3 area 0       ! g1/0 -> R8
R7(config-router)#network 10.7.7.7 0.0.0.0 area 0       ! lo0

! ROUTER 8 ----------------------------------------------------------------
R8(config-if)#router ospf 1
R8(config-router)#router-id 10.8.8.8
R8(config-router)#network 10.7.8.0 0.0.0.3 area 0       ! g1/0 -> R7
R8(config-router)#network 10.8.8.8 0.0.0.0 area 0       ! lo0
```

__AS 5511__ - FTRSI (Orange - Worldwide IP Backbone)

```txt
! ROUTER 10 ---------------------------------------------------------------
R10(config-if)#router ospf 1
R10(config-router)#router-id 10.10.10.10
R10(config-router)#network 10.10.11.0 0.0.0.3 area 0    ! g1/0 -> R11
R10(config-router)#network 10.10.12.0 0.0.0.3 area 1    ! g3/0 -> R12
R10(config-router)#area 1 stub no-summary               ! Totally Stub
R10(config-router)#network 10.10.10.10 0.0.0.0 area 0   ! lo0

! ROUTER 11 ---------------------------------------------------------------
R11(config-if)#router ospf 1
R11(config-router)#router-id 10.11.11.11
R11(config-router)#network 10.10.11.0 0.0.0.3 area 0  ! g1/0 -> R10
R11(config-router)#network 10.11.12.0 0.0.0.3 area 1  ! g2/0 -> R12
R11(config-router)#area 1 stub no-summary
R11(config-router)#network 10.11.11.11 0.0.0.0 area 0 ! lo0

! ROUTER 12 ---------------------------------------------------------------
R12(config-if)#router ospf 1
R12(config-router)#router-id 10.12.12.12
R12(config-router)#network 10.11.12.0 0.0.0.3 area 1  ! g2/0 -> R11
R12(config-router)#network 10.10.12.0 0.0.0.3 area 1  ! g3/0 -> R10
R12(config-router)#network 10.12.12.12 0.0.0.0 area 1 ! lo0
R12(config-router)#area 1 stub no-summary
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

            OSPF Router with ID (10.1.1.1) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        764         0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        952         0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        394         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        205         0x80000012 0x00D465 4
10.5.5.5        10.5.5.5        338         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        353         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        952         0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        1148        0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        957         0x8000000A 0x006184
10.3.4.2        10.4.4.4        1460        0x80000008 0x008063
10.3.5.1        10.3.3.3        394         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        205         0x80000008 0x00DDFA
            
R1#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/3] via 10.1.3.2, 03:53:04, GigabitEthernet2/0
                    [110/3] via 10.1.2.2, 03:53:14, GigabitEthernet1/0
O       10.2.2.2/32 [110/2] via 10.1.2.2, 03:53:14, GigabitEthernet1/0
O       10.3.3.3/32 [110/2] via 10.1.3.2, 03:53:04, GigabitEthernet2/0
O       10.6.6.6/32 [110/4] via 10.1.3.2, 03:42:56, GigabitEthernet2/0
                    [110/4] via 10.1.2.2, 03:42:56, GigabitEthernet1/0
O       10.3.5.0/30 [110/2] via 10.1.3.2, 03:53:04, GigabitEthernet2/0
O       10.2.4.0/30 [110/2] via 10.1.2.2, 03:53:14, GigabitEthernet1/0
O       10.3.4.0/30 [110/2] via 10.1.3.2, 03:53:04, GigabitEthernet2/0
O       10.4.4.4/32 [110/3] via 10.1.3.2, 03:53:04, GigabitEthernet2/0
                    [110/3] via 10.1.2.2, 03:53:14, GigabitEthernet1/0
O       10.5.5.5/32 [110/3] via 10.1.3.2, 03:46:47, GigabitEthernet2/0

R1#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.3.3.3          1   FULL/DR         00:00:31    10.1.3.2        GigabitEthernet2/0
10.2.2.2          1   FULL/DR         00:00:32    10.1.2.2        GigabitEthernet1/0

R1#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.1.1.1/32        1     LOOP  0/0
Gi2/0        1     0               10.1.3.1/30        1     BDR   1/1
Gi1/0        1     0               10.1.2.1/30        1     BDR   1/1

! ROUTER 2 --------------------------------------------------------------
R2#sh ip ospf database

            OSPF Router with ID (10.2.2.2) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        822         0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        1008        0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        452         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        261         0x80000012 0x00D465 4
10.5.5.5        10.5.5.5        396         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        409         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        1008        0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        1206        0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        1013        0x8000000A 0x006184
10.3.4.2        10.4.4.4        1516        0x80000008 0x008063
10.3.5.1        10.3.3.3        452         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        261         0x80000008 0x00DDFA
            
R2#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.4.4.4          1   FULL/DR         00:00:33    10.2.4.2        GigabitEthernet2/0
10.1.1.1          1   FULL/BDR        00:00:32    10.1.2.1        GigabitEthernet1/0

R2#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/2] via 10.2.4.2, 03:59:28, GigabitEthernet2/0
O       10.1.3.0/30 [110/2] via 10.1.2.1, 03:57:47, GigabitEthernet1/0
O       10.3.3.3/32 [110/3] via 10.2.4.2, 03:59:28, GigabitEthernet2/0
                    [110/3] via 10.1.2.1, 03:57:47, GigabitEthernet1/0
O       10.1.1.1/32 [110/2] via 10.1.2.1, 03:57:57, GigabitEthernet1/0
O       10.6.6.6/32 [110/3] via 10.2.4.2, 03:47:38, GigabitEthernet2/0
O       10.3.5.0/30 [110/3] via 10.2.4.2, 03:59:28, GigabitEthernet2/0
                    [110/3] via 10.1.2.1, 03:57:47, GigabitEthernet1/0
O       10.3.4.0/30 [110/2] via 10.2.4.2, 03:59:28, GigabitEthernet2/0
O       10.4.4.4/32 [110/2] via 10.2.4.2, 03:59:28, GigabitEthernet2/0
O       10.5.5.5/32 [110/4] via 10.2.4.2, 03:51:29, GigabitEthernet2/0
                    [110/4] via 10.1.2.1, 03:51:29, GigabitEthernet1/0
                    
R2#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.2.2.2/32        1     LOOP  0/0
Gi2/0        1     0               10.2.4.1/30        1     BDR   1/1
Gi1/0        1     0               10.1.2.2/30        1     DR    1/1

! ROUTER 3 --------------------------------------------------------------
R3#sh ip ospf database

            OSPF Router with ID (10.3.3.3) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        555         0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        743         0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        183         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        2023        0x80000011 0x00D664 4
10.5.5.5        10.5.5.5        127         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        142         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        743         0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        936         0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        746         0x8000000A 0x006184
10.3.4.2        10.4.4.4        1249        0x80000008 0x008063
10.3.5.1        10.3.3.3        183         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        2023        0x80000007 0x00DFF9
            
R3#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/2] via 10.3.4.2, 04:09:19, GigabitEthernet1/0
O       10.2.2.2/32 [110/3] via 10.3.4.2, 04:02:50, GigabitEthernet1/0
                    [110/3] via 10.1.3.1, 04:01:16, GigabitEthernet2/0
O       10.1.2.0/30 [110/2] via 10.1.3.1, 04:01:16, GigabitEthernet2/0
O       10.1.1.1/32 [110/2] via 10.1.3.1, 04:01:16, GigabitEthernet2/0
O       10.6.6.6/32 [110/3] via 10.3.4.2, 03:51:07, GigabitEthernet1/0
O       10.2.4.0/30 [110/2] via 10.3.4.2, 04:04:26, GigabitEthernet1/0
O       10.4.4.4/32 [110/2] via 10.3.4.2, 04:09:19, GigabitEthernet1/0
O       10.5.5.5/32 [110/2] via 10.3.5.2, 03:55:08, GigabitEthernet3/0

R3#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.5.5.5          1   FULL/BDR        00:00:33    10.3.5.2        GigabitEthernet3/0
10.1.1.1          1   FULL/BDR        00:00:35    10.1.3.1        GigabitEthernet2/0
10.4.4.4          1   FULL/DR         00:00:36    10.3.4.2        GigabitEthernet1/0

R3#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.3.3.3/32        1     LOOP  0/0
Gi3/0        1     0               10.3.5.1/30        1     DR    1/1
Gi2/0        1     0               10.1.3.2/30        1     DR    1/1
Gi1/0        1     0               10.3.4.1/30        1     BDR   1/1

! ROUTER 4 --------------------------------------------------------------
R4#sh ip ospf database

            OSPF Router with ID (10.4.4.4) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        882         0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        1068        0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        510         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        319         0x80000012 0x00D465 4
10.5.5.5        10.5.5.5        454         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        467         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        1068        0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        1263        0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        1071        0x8000000A 0x006184
10.3.4.2        10.4.4.4        1574        0x80000008 0x008063
10.3.5.1        10.3.3.3        510         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        319         0x80000008 0x00DDFA

R4#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.2.2.2/32 [110/2] via 10.2.4.1, 04:12:53, GigabitEthernet2/0
O       10.1.3.0/30 [110/2] via 10.3.4.1, 04:12:14, GigabitEthernet1/0
O       10.3.3.3/32 [110/2] via 10.3.4.1, 04:17:27, GigabitEthernet1/0
O       10.1.2.0/30 [110/2] via 10.2.4.1, 04:12:14, GigabitEthernet2/0
O       10.1.1.1/32 [110/3] via 10.3.4.1, 04:11:10, GigabitEthernet1/0
                    [110/3] via 10.2.4.1, 04:11:10, GigabitEthernet2/0
O       10.6.6.6/32 [110/2] via 10.4.6.2, 04:01:11, GigabitEthernet3/0
O       10.3.5.0/30 [110/2] via 10.3.4.1, 04:19:13, GigabitEthernet1/0
O       10.5.5.5/32 [110/3] via 10.3.4.1, 04:04:52, GigabitEthernet1/0

R4#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.6.6.6          1   FULL/BDR        00:00:32    10.4.6.2        GigabitEthernet3/0
10.2.2.2          1   FULL/BDR        00:00:32    10.2.4.1        GigabitEthernet2/0
10.3.3.3          1   FULL/BDR        00:00:31    10.3.4.1        GigabitEthernet1/0

R4#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.6.6.6          1   FULL/BDR        00:00:32    10.4.6.2        GigabitEthernet3/0
10.2.2.2          1   FULL/BDR        00:00:32    10.2.4.1        GigabitEthernet2/0
10.3.3.3          1   FULL/BDR        00:00:31    10.3.4.1        GigabitEthernet1/0
R4#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.4.4.4/32        1     LOOP  0/0
Gi3/0        1     0               10.4.6.1/30        1     DR    1/1
Gi2/0        1     0               10.2.4.2/30        1     DR    1/1
Gi1/0        1     0               10.3.4.2/30        1     DR    1/1

! ROUTER 5 --------------------------------------------------------------
R5#sh ip ospf database

            OSPF Router with ID (10.5.5.5) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        1100        0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        1288        0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        728         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        539         0x80000012 0x00D465 4
10.5.5.5        10.5.5.5        669         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        687         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        1288        0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        1481        0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        1290        0x8000000A 0x006184
10.3.4.2        10.4.4.4        1794        0x80000008 0x008063
10.3.5.1        10.3.3.3        728         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        539         0x80000008 0x00DDFA

R5#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.4.6.0/30 [110/3] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.2.2.2/32 [110/4] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.1.3.0/30 [110/2] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.3.3.3/32 [110/2] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.1.2.0/30 [110/3] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.1.1.1/32 [110/3] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.6.6.6/32 [110/4] via 10.3.5.1, 04:03:56, GigabitEthernet3/0
O       10.2.4.0/30 [110/3] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.3.4.0/30 [110/2] via 10.3.5.1, 04:07:51, GigabitEthernet3/0
O       10.4.4.4/32 [110/3] via 10.3.5.1, 04:07:51, GigabitEthernet3/0

R5#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.3.3.3          1   FULL/DR         00:00:31    10.3.5.1        GigabitEthernet3/0

R5#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.5.5.5/32        1     LOOP  0/0
Gi3/0        1     0               10.3.5.2/30        1     BDR   1/1

! ROUTER 6 --------------------------------------------------------------
R6#sh ip ospf database

            OSPF Router with ID (10.6.6.6) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.1        10.1.1.1        1326        0x8000000B 0x00B2F1 3
10.2.2.2        10.2.2.2        1512        0x8000000E 0x001281 3
10.3.3.3        10.3.3.3        954         0x8000000E 0x00E06F 4
10.4.4.4        10.4.4.4        763         0x80000012 0x00D465 4
10.5.5.5        10.5.5.5        898         0x8000000B 0x00960F 2
10.6.6.6        10.6.6.6        909         0x8000000A 0x00D7C1 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.2.2        10.2.2.2        1512        0x8000000A 0x0050A7
10.1.3.2        10.3.3.3        1707        0x8000000A 0x004BA5
10.2.4.2        10.4.4.4        1515        0x8000000A 0x006184
10.3.4.2        10.4.4.4        2018        0x80000008 0x008063
10.3.5.1        10.3.3.3        954         0x80000008 0x00C71C
10.4.6.1        10.4.4.4        763         0x80000008 0x00DDFA

R6#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O       10.2.2.2/32 [110/3] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.1.3.0/30 [110/3] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.3.3.3/32 [110/3] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.1.2.0/30 [110/3] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.1.1.1/32 [110/4] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.3.5.0/30 [110/3] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.2.4.0/30 [110/2] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.3.4.0/30 [110/2] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.4.4.4/32 [110/2] via 10.4.6.1, 04:07:38, GigabitEthernet3/0
O       10.5.5.5/32 [110/4] via 10.4.6.1, 04:07:38, GigabitEthernet3/0

R6#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.4.4.4          1   FULL/DR         00:00:36    10.4.6.1        GigabitEthernet3/0

R6#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.6.6.6/32        1     LOOP  0/0
Gi3/0        1     0               10.4.6.2/30        1     BDR   1/1
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
R7#sh ip ospf database

            OSPF Router with ID (10.7.7.7) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.7.7.7        10.7.7.7        431         0x8000000A 0x008304 2
10.8.8.8        10.8.8.8        530         0x8000000A 0x0082FA 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.7.8.1        10.7.7.7        431         0x80000008 0x0004B7

R7#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O       10.8.8.8/32 [110/2] via 10.7.8.2, 04:02:17, GigabitEthernet1/0

R7#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.8.8.8          1   FULL/BDR        00:00:32    10.7.8.2        GigabitEthernet1/0

R7#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.7.7.7/32        1     LOOP  0/0
Gi1/0        1     0               10.7.8.1/30        1     DR    1/1

! ROUTER 8 --------------------------------------------------------------
R8#sh ip ospf database

            OSPF Router with ID (10.8.8.8) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.7.7.7        10.7.7.7        599         0x8000000A 0x008304 2
10.8.8.8        10.8.8.8        696         0x8000000A 0x0082FA 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.7.8.1        10.7.7.7        599         0x80000008 0x0004B7

R8#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O       10.7.7.7/32 [110/2] via 10.7.8.1, 04:05:14, GigabitEthernet1/0

R8#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.7.7.7          1   FULL/DR         00:00:31    10.7.8.1        GigabitEthernet1/0

R8#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.8.8.8/32        1     LOOP  0/0
Gi1/0        1     0               10.7.8.2/30        1     BDR   1/1
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
R10#sh ip ospf database

            OSPF Router with ID (10.10.10.10) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     1469        0x8000000A 0x00441B 2
10.11.11.11     10.11.11.11     1355        0x8000000A 0x004312 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.11.1      10.10.10.10     1469        0x80000007 0x004853

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.0      10.10.10.10     1717        0x80000007 0x00C828
10.10.12.0      10.11.11.11     1107        0x80000007 0x00BD2F
10.11.12.0      10.10.10.10     965         0x80000007 0x00C628
10.11.12.0      10.11.11.11     1355        0x80000007 0x00A745
10.12.12.12     10.10.10.10     965         0x80000007 0x00548A
10.12.12.12     10.11.11.11     1107        0x80000007 0x003F9C

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     965         0x80000008 0x00ACEC 1
10.11.11.11     10.11.11.11     1117        0x80000008 0x008A07 1
10.12.12.12     10.12.12.12     1093        0x8000000B 0x007D77 3

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.2      10.12.12.12     1093        0x80000007 0x00365C
10.11.12.2      10.12.12.12     1093        0x80000007 0x00513D

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     1726        0x80000007 0x007897
0.0.0.0         10.11.11.11     1365        0x80000007 0x0063A9

R10#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O       10.11.11.11/32 [110/2] via 10.10.11.2, 03:42:19, GigabitEthernet1/0
O       10.12.12.12/32 [110/2] via 10.10.12.2, 03:37:51, GigabitEthernet3/0
O       10.11.12.0/30 [110/2] via 10.10.12.2, 03:37:51, GigabitEthernet3/0

R10#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.11.11.11       1   FULL/BDR        00:00:31    10.10.11.2      GigabitEthernet1/0
10.12.12.12       1   FULL/DR         00:00:39    10.10.12.2      GigabitEthernet3/0

R10#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.10.10.10/32     1     LOOP  0/0
Gi1/0        1     0               10.10.11.1/30      1     DR    1/1
Gi3/0        1     1               10.10.12.1/30      1     BDR   1/1

! ROUTER 11 -------------------------------------------------------------
R11#sh ip ospf database

            OSPF Router with ID (10.11.11.11) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     1696        0x8000000A 0x00441B 2
10.11.11.11     10.11.11.11     1581        0x8000000A 0x004312 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.11.1      10.10.10.10     1696        0x80000007 0x004853

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.0      10.10.10.10     1945        0x80000007 0x00C828
10.10.12.0      10.11.11.11     1333        0x80000007 0x00BD2F
10.11.12.0      10.10.10.10     1192        0x80000007 0x00C628
10.11.12.0      10.11.11.11     1581        0x80000007 0x00A745
10.12.12.12     10.10.10.10     1193        0x80000007 0x00548A
10.12.12.12     10.11.11.11     1333        0x80000007 0x003F9C

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     1194        0x80000008 0x00ACEC 1
10.11.11.11     10.11.11.11     1334        0x80000008 0x008A07 1
10.12.12.12     10.12.12.12     1312        0x8000000B 0x007D77 3

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.2      10.12.12.12     1312        0x80000007 0x00365C
10.11.12.2      10.12.12.12     1312        0x80000007 0x00513D

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     1947        0x80000007 0x007897
0.0.0.0         10.11.11.11     1582        0x80000007 0x0063A9

R11#sh ip route ospf
     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O       10.10.10.10/32 [110/2] via 10.10.11.1, 03:46:09, GigabitEthernet1/0
O       10.12.12.12/32 [110/2] via 10.11.12.2, 03:41:19, GigabitEthernet2/0
O       10.10.12.0/30 [110/2] via 10.11.12.2, 03:41:19, GigabitEthernet2/0

R11#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.10.10       1   FULL/DR         00:00:33    10.10.11.1      GigabitEthernet1/0
10.12.12.12       1   FULL/DR         00:00:38    10.11.12.2      GigabitEthernet2/0

R11#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.11.11.11/32     1     LOOP  0/0
Gi1/0        1     0               10.10.11.2/30      1     BDR   1/1
Gi2/0        1     1               10.11.12.1/30      1     BDR   1/1

! ROUTER 12 -------------------------------------------------------------
R12#sh ip ospf database

            OSPF Router with ID (10.12.12.12) (Process ID 1)

		Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.10.10.10     10.10.10.10     1337        0x80000008 0x00ACEC 1
10.11.11.11     10.11.11.11     1479        0x80000008 0x008A07 1
10.12.12.12     10.12.12.12     1454        0x8000000B 0x007D77 3

		Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.12.2      10.12.12.12     1454        0x80000007 0x00365C
10.11.12.2      10.12.12.12     1454        0x80000007 0x00513D

		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.10.10.10     93          0x80000008 0x007698
0.0.0.0         10.11.11.11     1727        0x80000007 0x0063A9

R12#sh ip route ospf
O*IA 0.0.0.0/0 [110/2] via 10.11.12.1, 03:43:47, GigabitEthernet2/0
               [110/2] via 10.10.12.1, 03:43:37, GigabitEthernet3/0
               
R12#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.10.10       1   FULL/BDR        00:00:35    10.10.12.1      GigabitEthernet3/0
10.11.11.11       1   FULL/BDR        00:00:33    10.11.12.1      GigabitEthernet2/0

R12#sh ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     1               10.12.12.12/32     1     LOOP  0/0
Gi3/0        1     1               10.10.12.2/30      1     DR    1/1
Gi2/0        1     1               10.11.12.2/30      1     DR    1/1
```

#### Test the interfaces connectivity between the ASes

__Example from AS 1273 to AS 17390__

| From | To | Interface | IP Address | Connectivity Success |
| ---- | -- | --------- | ---------- | ------- |
| R1   | R8 | Lo1 | 130.41.46.88     | No      |
| R1   | R8 | g4/0 | 48.73.240.2     | Yes     |
| R1   | R8 | g5/0 | 130.41.46.2     | No      |
| R1   | R7 | Lo1  | 130.41.46.77    | No |
| R1   | R7 | f0/0 | 130.41.46.9     | No |
| R1   | R7 | g4/0 | 48.73.240.6     | No |
| R1   | R7 | g5/0 | 130.41.46.2     | No |

With the exception of the direct links between routers of distinct ASes, we have no connectivity. To have connectivity between ASes we need to configure BGP

__TODO__

### 1.4 Practical Questions

#### 1.4.1 - Create a comprehensive table presenting all the connectivity tests carried out and the respective outcome (e.g., success, failure). You don’t need to provide exhaustive snapshots for all test results. Choose only **three example cases**, from different ASes, to include in your report and briefly comment on each selected case.

__TODO__

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

__TODO__

#### 1.4.5 - Table showcasing all the peering relations established in the provided topology

__TODO__

#### 1.4.6 - Explain how a Tier-2 benefits from peering instead of buying everything from a Tier-1

__TODO__

#### 1.4.7 - Identify the neutral public peering interconnections in this lab topology. Elaborate on why they are called neutral and provide examples of real-world implementations of such public interconnections

__TODO__

#### 1.4.8 - Explain the role of R12 in AS 5511, and how are its interfaces divided between the OSPF areas involved

__TODO__

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

__TODO__

<div style="page-break-after: always"></div>

## 2 - Phase 2 - iBGP and eBGP Without Routing Policies

### 2.1 - Objectives

In this phase we have as objectives:

- Establish eBGP sessions between the AS’s
- Inside the AS 1273, establish iBGP sessions between the clients and the two route reflectors, R3 and R4 to avoid the full mesh
- Server subnet public IPs are listed in the internet rouXng table
- Implement connecXvity between the ASes from any routers using the Lo1 and from any server

### 2.2 - Implementation

__TODO__

### 2.3 - Test and Validation

__TODO__

### 2.4 - Practical Questions

#### 2.4.1 - How is the BGP next hop reachability solved inside an AS?

__TODO__  next-hop self????

#### 2.4.2 - Why is it a good pracXce to use the loopback IP address in the iBGP sessions? 

__TODO__  Resiliency in case of phisical ports went down?

#### 2.4.3 - Create and present a detailed table with all the connectivity tests performed using the TCLSH procedure, as previously outlined

__TODO__

#### 2.4.4 - In the following Local-RIB table output example, how many routes were installed for the destination 46.87.162.0/24? Justify your response explaining the decision process in BGP

```txt
     Network         Next Hop       Metric LocPrf Weight Path
* 46.87.162.0/24     130.41.46.5                       0 17390 1273 20717 5511 i
*>                   130.41.46.9                       0 17390 1273 20717 5511 i
```

__TODO__

#### 2.4.5 - What would have happened in case you didn’t configure the next-hop-self on the iBGP peering definitions? What reasons explain why BGP doesn’t set the next-hop-self as a default setting?

__TODO__

#### 2.4.6 - How are the route prefixes propagated inside the AS when you have route reflectors configured?

__TODO__

#### 2.4.7 - Simulate and explain using Wireshark the BGP messages associated to each BGP state (see: [article](https://www.ciscopress.com/articles/article.asp?p=2756480&seqNum=4) )

__TODO__

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

__TODO__

### 3.3 - Test and Validation

__TODO__

### 3.4 - Practical Questions

#### 3.4.1 - After applying your prefix-list or route-map, compare the output of `show ip bgp` and `show ip bgp neighbors <ip> advertised-routes`. Explain the differences you observe referring to the __Adj-RIB-In__, __Adj-RIB-Out__, and __Loc-RIB__ tables

__TODO__

#### 3.4.2 - Why is the control from the number of prefixes advertised to the internet a good practice?

__TODO__

#### 3.4.3 - What is the private AS number range in BGP? Describe some scenarios where using private ASNs can be useful.

Private ASN Ranges:

| Format | Private Range  |
| ------ | -------------- |
| 2-byte | 64512 to 65534 |
| 4-byte | 4200000000 to 4294967294 |

__TODO__

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

__TODO__

#### 3.4.5 - Imagine that one of your prefixes was not selected as the best path in your lab. Based on the __BGP decision process__, propose a configuration change (e.g., adjusting local preference, MED, or weight) that would alter the selection. Justify your answer by showing the relevant command(s) and predicting the impact on the routing table.

__TODO__

<div style="page-break-after: always"></div>

## 4 - Phase 4 - Influence the Internet Routing

### 4.1 - Objectives

In this phase the objectives are:

- Enable authenXcaXon on eBGP peerings in __AS701__
- Filter Bogon prefixes received from the _“BadGuy”_ router
- Apply __Remote Triggered Black Hole (RTBH)__ filtering to mitigate a DoS attack within __AS1273__

### 4.2 - Implementation

__TODO__

### 4.3 - Test and Validation

__TODO__

### 4.4 - Practical Questions

#### 4.4.1 - Describe in your report the policy options you used to implement the routing policies (include all the details of the configurations (e.g., prefix-list, route-map, etc)

__TODO__

#### 4.4.2 - Provide command output screenshots that demonstrate the successful application of the configured policies

__TODO__

#### 4.4.3 - Discuss other alternative to achieve the same results and comment on their relative pros and cons, compared to your implementation

__TODO__

<div style="page-break-after: always"></div>

## 5 - Phase 5 - Security Practices

### 5.1 Objectives

In this last phase, the objectives are:

- Activate the MD5 authentication on all the eBGP peering’s on the AS701. This a good practice for all the peers nevertheless for the lab proposes will be ok to test on this AS only
- Filter the Bogon prefixes on the peer routers from AS 1273. The BadGuy router is adverXsing Bogons (see: [link](https://conference.apnic.net/22/docs/tut-routing-pres-bgp-bcp.pdf))
- Implement Remote Triggered Black Hole (RTBH) filtering - a popular and effective technique for the mitigation of denial-of-service (DoS) attack on AS1273 coming from the prefix `63.96.0.115` [link](https://www.cisco.com/c/dam/en_us/about/security/intelligence/blackhole)

### 5.2 - Implementation

__TODO__

### 5.3 - Test and Validation

__TODO__

### 5.4 - Practical Questions

#### 5.4.1 - How does MD5 authentication improve the security of BGP sessions? What happens if it is not enabled?

__TODO__

#### 5.4.2 - Which Bogon ranges did you filter in your lab, and why must they not appear in the global routing table?

__TODO__

#### 5.4.3 - Explain how RTBH was implemented in AS1273. Include a diagram showing how the trigger router propagates the blackhole route

__TODO__

#### 5.4.4 - Describe the role of uRPF in validating traffic source addresses and prevenXng spoofing

__TODO__

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