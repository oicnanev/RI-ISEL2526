# LAB Project Nº 1 - VLAN/STP/OSPF MONO-AREA

> REDES DE INTERNET (RI)2025-2026

---

## Group 1 - Class 52D??? (not sure)

### Professor Luís Mata 

> luis.mata@isel.pt

- 45824 Nuno Venâncio
- 49420 André Carrilho
- 51454 Hugo Leal

---

## Index

- Introduction
- 1 - Enterprise A — L2 Foundations (VLAN/STP/RSTP)
    - 1.1 Implementation
        - 1.1.1 VLANs
        - 1.1.2 STP
    - 1.2 Tests and Validation
    - 1.3 Practical Questions
- 2 - Enterprise A — VLAN Segmentation & Addressing
    - 2.1 Implementation
        - 2.1.1 VLANs
        - 2.1.2 Access and Trunk Ports
        - 2.1.3 PCs IP Assignments
    - 2.2 Test and Validation
    - 2.3 Practical Questions
- 3 - Enterprise A — Router-on-a-Stick (RoS) & L3 Rules (no ACLs)
    - 3.1 Implementation
        - 3.1.1 Creation and configuration of sub interfaces to implement the L3 (Rules without ACLs)
        - 3.1.2 Trunk link between Router A and SW_DC
        - 3.1.3 Sub interfaces with encapsulation dot1Q and IPs (parent interface no IP)
    - 3.2 Test and Validation
    - 3.3 Practical Questions 
- 4 - Enterprise B — Segmentation & ISP L2 Interconnection
    - 4.1 Implementation
        - 4.1.1 Company B VLANs and IP Assignments
        - 4.1.2 Implementation of the ISP topology for interconnection with customers
        - 4.1.3 Building of VLAN paths in the switch fabric
        - 4.1.4 Assignment of IP address to Router 1, Router 3, Router A, and Router B
    - 4.2 Test and Validation
    - 4.3 Practical Questions
- 5 - Static Routing → OSPF Core & “Internet” Loopback → Public Addressing Test
    - 5.1 Implementation
        - 5.1.1 Static routing (initial)
            - 5.1.1.1 Default static routes on company routers toward the ISP; identify next-hop and purpose.
            - 5.1.1.2 Static routing on R1–R4, add required static routes to reach company blocks.
            - 5.1.1.3 Verification of which pings succeed before OSPF and why.
        - 5.1.2 OSPF (ISP core)
            - 5.1.2.1 Configuration of OSPF on R1–R4 (single area) per design
            - 5.1.2.2 Internet Simulation - Configuration of R2 Loopback0 = 8.8.8.8/32
            - 5.1.2.3 Global connectivity with redundancy
        - 5.1.3 Public addressing test
            - 5.1.3.1 Change of configs so each company and the operator use public IPs
    - 5.2 Test and Validation
    - 5.3 Practical Questions
- Conclusion

---

## Introduction

In this project we explore Internet Networks topics like VLAN segmentation, L2 loop protection (STP/RSTP), inter-VLAN routing (router-on-a-stick), static routing, and single-area OSPF.

To accomplish this we will configure and simulate a network between two enterprises (A and B) and an ISP (Internet Service Provider).

The Enterprise A will be constituted of 1 router, 5 switches, 5 PCs and 4 VLANs.... __TODO__

The Enterprise B will be constituted of 1 router, 1 switch, 1 server, 1 pc and 2 VLANs.... __TODO__

The ISP will be constituted by 4 routers and 4 switches, that connects to "Internet" using a Loopback.... __TODO__

__TODO__

![VLAN/STP/OSPF MONO-AREA](./assets/01.png)

___

## 1. Enterprise A - L2 Foundations (VLAN/STP/RSTP)

### 1.1 Implementation

__TODO__

#### 1.1.1 VLANs

__TODO__

| VLAN | NAME             | IP GATEWAY    | NETWORK          | PCs |
| -- | ------------------ | ------------- | -----------------| -------- |
| 11 | Accounting         | 172.20.11.254 | 172.20.11.0/24   | PC7, PC9 |
| 12 | Secretariat        | 172.20.12.254 | 172.20.12.0/24   | PC5, PC8 |
| 13 | Computer science   | 172.20.13.126 | 172.20.13.0/25   | PC6 |

#### 1.1.2 STP (RB/roles/costs)

__TODO__

- Table built using command `show spanning-tree`

| Switch    | VLAN #   | Priority | MAC            | Root MAC       |
| --------- | -------- | -------- | -------------- | -------------- |
| SW_DC     | VLAN0001 | 32798 | 0002.4AE4.E093 | 00E0.A3CE.4A46 (sw1_piso2) |
| SW_DC     | VLAN0020 | 32788 | 0002.4AE4.E093 | Same |
| sw1_piso1 | VLAN0001 | 32769 | 0009.7C65.BA4B | 00E0.A3CE.4A46 (sw1_piso2) |
| sw1_piso1 | VLAN0020 | 32788 | 0009.7C65.BA4B | 0002.4AE4.E093 (SW_DC) |
| sw1_piso1 | VLAN0030 | 32798 | 0009.7C65.BA4B | 0002.4AE4.E093 (SW_DC) |
| sw1_piso1 | VLAN0040 | 32808 | 0009.7C65.BA4B | 0002.4AE4.E093 (SW_DC) |
| sw1_piso1 | VLAN0045 | 32813 | 0009.7C65.BA4B | 0002.4AE4.E093 (SW_DC) |
| sw2_piso1 | VLAN0001 | 32769 | 00E0.F950.631A | 00E0.A3CE.4A46 (sw1_piso2) |
| sw2_piso1 | VLAN0020 | 32788 | 00E0.F950.631A | 0002.4AE4.E093 (SW_DC) |
| sw2_piso1 | VLAN0030 | 32798 | 00E0.F950.631A | 0002.4AE4.E093 (SW_DC) |
| sw2_piso1 | VLAN0040 | 32808 | 00E0.F950.631A | 0002.4AE4.E093 (SW_DC) |
| sw2_piso1 | VLAN0045 | 32813 | 00E0.F950.631A | 0002.4AE4.E093 (SW_DC) |
| sw1_piso2 | VLAN0001 | 28673 | 00E0.A3CE.4A46 | Same |
| sw1_piso2 | VLAN0020 | 32788 | 00E0.A3CE.4A46 | 0002.4AE4.E093 (SW_DC) |
| sw1_piso2 | VLAN0030 | 32798 | 00E0.A3CE.4A46 | 0002.4AE4.E093 (SW_DC) |
| sw1_piso2 | VLAN0040 | 32808 | 00E0.A3CE.4A46 | 0002.4AE4.E093 (SW_DC) |
| sw1_piso2 | VLAN0045 | 32813 | 00E0.A3CE.4A46 | 0002.4AE4.E093 (SW_DC) |
| sw2_piso2 | VLAN0001 | 32769 | 00E0.8FE1.53D7 | 00E0.A3CE.4A46 (sw1_piso2) |
| sw2_piso2 | VLAN0020 | 32788 | 00E0.8FE1.53D7 | 0002.4AE4.E093 (SW_DC) |
| sw2_piso2 | VLAN0030 | 32798 | 00E0.8FE1.53D7 | 0002.4AE4.E093 (SW_DC) |
| sw2_piso2 | VLAN0040 | 32808 | 00E0.8FE1.53D7 | 0002.4AE4.E093 (SW_DC) |
| sw2_piso2 | VLAN0045 | 32813 | 00E0.8FE1.53D7 | 0002.4AE4.E093 (SW_DC) |

- __VLAN0001__

| Port             | PC | RPC | RP | DP | ALT/BLK |
| ---------------- | -- | --- | -- | -- | ------- |
| sw1_piso2 Fa0/2  | 19 | -   |    | X  |         |
| sw1_piso2 Fa0/18 | 19 | -   |    | X  |         |
| sw1_piso2 Fa0/23 | 19 | -   |    | X  |         |
| sw1_piso2 Fa0/24 | 19 | -   |    | X  |         | 

```shell
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 100       128.2    P2p
Fa0/18           Desg FWD 19        128.18   P2p
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
```


### 1.2 Tests and Validation

__TODO__

#### Summary of Spanning-Tree by Switch

> `show spanning-tree summary`

- __SW_DC__

```shell
Switch is in rapid-pvst mode
Root bridge for: Contabilidade Secretariado Informatica Gestao_de_rede
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     1         0        0          2          3
VLAN0020                     0         0        0          3          3
VLAN0030                     0         0        0          3          3
VLAN0040                     0         0        0          3          3
VLAN0045                     0         0        0          3          3

---------------------- -------- --------- -------- ---------- ----------
5 vlans                      1         0        0         14         15
```

- __sw1_piso1__

```shell
Switch is in rapid-pvst mode
Root bridge for:
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     2         0        0          3          5
VLAN0020                     1         0        0          4          5
VLAN0030                     0         0        0          5          5
VLAN0040                     1         0        0          4          5
VLAN0045                     1         0        0          4          5

---------------------- -------- --------- -------- ---------- ----------
5 vlans                      5         0        0         20         25
```

- __sw2_piso2__

```shell
Switch is in rapid-pvst mode
Root bridge for:
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     2         0        0          4          6
VLAN0020                     4         0        0          2          6
VLAN0030                     4         0        0          2          6
VLAN0040                     3         0        0          3          6
VLAN0045                     3         0        0          3          6

---------------------- -------- --------- -------- ---------- ----------
5 vlans                     16         0        0         14         30
```

- __sw1_piso1__

```shell
Switch is in rapid-pvst mode
Root bridge for: default
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     1         0        0          4          5
VLAN0020                     1         0        0          4          5
VLAN0030                     2         0        0          3          5
VLAN0040                     3         0        0          2          5
VLAN0045                     2         0        0          3          5

---------------------- -------- --------- -------- ---------- ----------
5 vlans                      9         0        0         16         25
```

- __sw2_piso2__

```shell
Switch is in rapid-pvst mode
Root bridge for:
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     4         0        0          1          5
VLAN0020                     3         0        0          2          5
VLAN0030                     3         0        0          2          5
VLAN0040                     4         0        0          1          5
VLAN0045                     4         0        0          1          5

---------------------- -------- --------- -------- ---------- ----------
5 vlans                     18         0        0          7         25
```


### 1.3 Practical Questions

#### 1. What is the purpose of this command - "no ip domain-lookup"?

__TODO__

#### 2. What default VLANs exist before any VLAN is configured on any equipment?

__TODO__

#### 3. What is the format of the tags introduced in Ethernet frames in trunk connections?

__TODO__

#### 4. Why is it that on a LAN that uses VLANs, on an Access-type connection the frames do not include tags?

__TODO__

#### 5. What is the tag that the wefts belonging to VLAN 1 carry?

__TODO__

#### 6. When a machine receives an Ethernet frame, how does it differ if it includes the Type/Length field after the source address field or if it includes the fields associated with a VLAN?

__TODO__

#### 7. What are the possible consequences of passing the timers "Max Age"=20 sec and "Forward Delay"= 15 sec to half of these values?

__TODO__

#### 8. What is the Root Bridge (RB)? Justify.

__TODO__ 

#### 9. By default, what is the type of active Spanning-Tree (STP) [hint: use sh span]? 

__TODO__

#### 10. How many spanning trees are there in the implemented topology? 

__TODO__

#### 11.  For company A, build the table for calculating the cost of the paths and determining which are the Root, Designated and Blocking ports and calculate the respective values (check in the PT which cost values are used in the calcula[ons of the spanning trees). Are the final results you arrived at consistent with those that the PT simulator presents?

__TODO__

#### 12. What is the cost of the shortest path to Router A since PC9? 

__TODO__

---

## 2. Enterprise A - VLAN Segmentation and Addressing

### 2.1 Implementation

__TODO__

#### 2.1.1 VLANs

__TODO__: add table and show configurations

> O trabalho não tem Server2 !!?

| VLAN | NAME             | IP GATEWAY    | NETWORK          | PCs |
| -- | ------------------ | ------------- | -----------------| -------- |
| 11 | Accounting         | 172.20.11.254 | 172.20.11.0/24   | PC7, PC9 |
| 12 | Secretariat        | 172.20.12.254 | 172.20.12.0/24   | PC5, PC8 |
| 13 | Computer science   | 172.20.13.126 | 172.20.13.0/25   | PC6 |
| 14 | Network management | 172.20.13.254 | 172.20.13.127/25 | Server2 |

#### 1.1.2 Access and Trunk Ports

__TODO__: maybe a table

#### 2.1.3 PCs IP Assignments

__TODO__: maybe a table

### 2.2 Test and Validation

__TODO__

### 2.3 Practical Questions

#### 1. What is the cost of the shortest path to Router A since PC9? 

__TODO__

#### 2. Explain what ac[ons were required to accommodate the topology requirements.

__TODO__

#### 3. Did you achieve inter vlan connectivity in this phase? Explain the observed behaviour.

---
## 3. Enterprise A — Router-on-a-Stick (RoS) & L3 Rules (no ACLs)

### 3.1 Implementation

__TODO__

#### 3.1.1 Creation and configuration of sub interfaces to implement the L3 (Rules without ACLs)

- Accounting & Secretariat: no access to any other internal or external VLAN/network
- IT: access to Secretariat and outside Company A
- Net-Management: mutual access (all support equipment must be reachable from PC6 for remote management)

__TODO__

#### 3.1.2 Trunk link between Router A and SW_DC

__TODO__

#### 3.1.3 Sub interfaces with encapsulation dot1Q and IPs (parent interface no IP)

__TODO__

### 3.2 Test and Validation

__TODO__

### 3.3 Practical Questions

#### 1. Explain the advantages and disadvantages of the Router-on-a-stick functionality

__TODO__

#### 2. Explain how you enforced the three communication rules __without ACLs__

__TODO__

#### 3. Provide short command outputs proving each rule is satisfied/blocked as required

__TODO__

---

## 4. Enterprise B — Segmentation & ISP L2 Interconnection

### 4.1 Implementation

#### 4.1.1 Company B VLANs and IP Assignments

__TODO__

#### 4.1.2 Implementation of the ISP topology for interconnection with customers

__TODO__

#### 4.1.3 Building of VLAN paths in the switch fabric

__TODO__

#### 4.1.4 Assignment of IP address to Router 1, Router 3, Router A, and Router B

__TODO__

### 4.2 Test and Validation

__TODO__

### 4.3 Practical Questions

#### 1. Count STP trees; list blocked ports in VLANs 90 and 95 and justify.

__TODO__

#### 2. State the advantage/goal of trunk pruning in the ISP fabric.

__TODO__

#### 3. Explain the chosen RB priori[es and observed blocked ports.

__TODO__

---

## 5. Static Routing → OSPF Core & “Internet” Loopback → Public Addressing Test

### 5.1 Implementation

#### 5.1.1 Static routing (initial)

##### 5.1.1.1 Default static routes on company routers toward the ISP; identify next-hop and purpose.

__TODO__

##### 5.1.1.2 Static routing on R1–R4, add required static routes to reach company blocks.

__TODO__

##### 5.1.1.3 Verification of which pings succeed before OSPF and why.

__TODO__

#### 5.1.2 OSPF (ISP core)

##### 5.1.2.1 Configuration of OSPF on R1–R4 (single area) per design

__TODO__

##### 5.1.2.2 Internet Simulation - Configuration of R2 Loopback0 = 8.8.8.8/32

__TODO__

##### 5.1.2.3 Global connectivity with redundancy

__TODO__

#### 5.1.3 Public addressing test

##### 5.1.3.1 Change of configs so each company and the operator use public IPs

__TODO__

### 5.2 Testing and Validation

__TODO__

### 5.3 Practical Questions

#### 1. Can corporate PCs ping the ISP router before OSPF? Justify.

__TODO__

#### 2. What is the __next-hop__ of your default routes and why?

__TODO__

#### 3. Describe the purpose of static routes on R1–R4 and what changes once OSPF runs. 

__TODO__

#### 4. After enabling OSPF, explain the path selection you observe towards __8.8.8.8.__

__TODO__

---

## Conclusion

__TODO__



