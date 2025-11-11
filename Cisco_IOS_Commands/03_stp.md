# Spanning Tree Protocol (STP)

## Manually Change Switch Priority

```txt
SW(config)# spanning-tree vlan 1 priority 100
% Bridge Priority must be increments of 4096.
% Allowed values are:
0	4096	 8192	 12288	16384	20480	24576	28672
32768	36864	40960	45056	49152	53248	57344	61440
```

## View Spanning-Tree configurations

```txt
SW# show spanning-tree

! by VLAN
SW# show spanning-tree vlan 1
```

## Configuration of distinct default gateways on the root bridge of each VLAN

```txt
! For VLAN 1
S1(config)# interface vlan 1
S1(config)# ip address 192.168.1.254 255.255.255.0
S1(config)# no shutdown

! For VLAN 2
S1(config)# interface vlan 2
S1(config)# ip address 192.168.2.254 255.255.255.0
S1(config)# no shutdown

! For VLAN 3
S1(config)# interface vlan 3
S1(config)# ip address 192.168.3.254 255.255.255.0
S1(config)# no shutdown

S1(config)# exit
```



