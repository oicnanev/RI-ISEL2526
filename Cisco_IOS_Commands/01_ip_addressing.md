# IP Addressing

## Check Switch Address Table

```txt
Switch# show mac address-table dynamic

		Mac Address Table
---------------------------------------------

Vlan		Mac Address		Type		Ports
----		----------		-------		-----
  1		000c.853d.9ec1		DYNAMIC		Fa0/4
  1		0010.1128.4232		DYNAMIC 	Fa0/2
  1		0060.70d2.a343		DYNAMIC		Fa0/3
  1		0090.2b09.0ea7		DYNAMIC		Fa0/1   
```

## IP Routing Table

```txt
RouterA# show ip route
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 10.20.1.1 to network 0.0.0.0

     10.0.0.0/30 is subnetted, 1 subnets
C       10.20.1.0 is directly connected, FastEthernet0/0
     172.20.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.20.12.0/24 is directly connected, FastEthernet0/1.12
C       172.20.13.0/25 is directly connected, FastEthernet0/1.13
S*   0.0.0.0/0 [1/0] via 10.20.1.1
```

## Assign IP to Interface

```txt
RouterA(config)# int g0/1
RouterA(config-if)# ip address 192.168.1.24 255.255.255.0  
RouterA(config-if)# no shutdown
```

## Static Route Configuration

```txt
Router1(config) ip route 0.0.0.0 0.0.0.0 10.12.10.2             ! Gateway last resort
Router1(config) ip route 172.20.11.0 255.255.255.0 10.20.1.2    ! Company A (VLAN 11)
Router1(config) ip route 172.20.12.0 255.255.254.0 10.20.1.2    ! Company A (VLAN 12, 13)
Router1(config) ip route 192.168.1.0 255.255.255.224 10.14.10.2 ! Company B (VLAN 2)
Router1(config) ip route 192.168.2.0 255.255.255.224 10.14.10.2 ! Company B (VLAN 3) 
```