# Switching and VLANs

## Access Port Configuration

```txt
Switch(config)# vlan 10
Switch(config-vlan)# name ACCOUNTING
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# switchport nonegotiate
```

## Trunk Port Configuration

```txt
Switch(config)# interface fa0/24
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# switchport nonegotiate
```

> __note__: nonegotiate disables Dynamic Trunking Protocol (DTP), useful for static trunk setups

## VLAN Configuration and Status Checks

```txt
! Show VLANs and port assignments
Switch # show vlan brief

! Check trunk port status and allowed VLANs
Switch # show interfaces trunk

! Verify switchport mode and VLAN on an interface
Switch # show interfaces fa0 /24 switchport

! View the current config of a specific interface
Switch # show running - config interface fa0 /24

! Confirm port status and VLAN assignment
Switch # show interfaces status
```

## VLAN Connectivity and Forwarding Checks

```txt
! Display MAC addresses learned in a VLAN
Switch # show mac address - table vlan 10

! Inspect STP role and status for a VLAN
Switch # show spanning - tree vlan 10

! Test connectivity between VLANs or devices
Switch # ping 192.168.10.1
Switch # traceroute 192.168.10.1
``` 

## Router-on-a-Stick (ROAS)

```txt
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 172.16.10.1 255.255.255.0
Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 172.16.20.1 255.255.255.0  
```

## Assign native VLAN to other than VLAN 1 (Security Reasons)

```txt
Switch(config) vlan 99
Switch(config) interfaces range [desired interfaces]
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan add 99
Switch(config-if)# switchport nonegociate
```

## Create and Name VLANs

```txt
Switch(config)# vlan 11
Switch(config-vlan)# name Accounting
Switch(config)# vlan 12
Switch(config-vlan)# name Secretariat
Switch(config)# vlan 13
Switch(config-vlan)# name Computer_science
```

