# Advanced STP

## Convert STP to Rapid PVST

```txt
SWITCH(config)# spanning-tree mode rapid-pvst
```

## Convert Access Ports to PortFast with BPDU Guard

```txt
! Enable PortFast and bpdu guard
sw1(config)# interface fa0/10
sw1(config-if)# spanning-tree portfast
sw1(config-if)# spanning-tree bpduguard enable

! Verify PortFast status
sw1# show spanning-tree interface fa0/10 portfast

! Confirm operational spanning-tree role and state
sw1# show spanning-tree interface fa0/10 detail
```

## Multiple Spanning Protocol

```txt
sw1(config)# spanning-tree mode mst
sw1(config)# spanning-tree mst configuration
sw1(config-mst)# name NAME
sw1(config-mst)# revision NUMBER
sw1(config-mst)# instance NUMBER vlan VLAN-LIST 
```

- NAME - name that identifies a set of parameters
- REVISION - number that identifies a set of parameters
- INSTANCE mapping - mapping of VLANs by switch (eg. `instance1 VLAN 1-30`)

