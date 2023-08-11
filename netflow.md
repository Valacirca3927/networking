## Flexible Netflow
 1. Records - Custom fields. You can match on whatever you want.
 1. Export Entities.
 1. Flow Monitor.

## AN IOS-XR Config

### Flow Exporter, Monitor Map and Sampler Map
```
flow exporter-map Destination_1
version v9
options interface-table
template data timeout 600
!
dscp 48
transport udp 2055
source Loopback1
destination <IP 1>
!
flow monitor-map internet
record ipv4
exporter Destination_1
cache timeout active 60
cache timeout inactive 5
!
sampler-map internet
random 1 out-of 500
!
Interface ten 3/1
flow ipv4 monitor internet sampler internet ingress
flow ipv4 monitor internet sampler internet egress

```

### Netflow via Policy-Maps
```
ip access-list extended ACL-JUST-R1
 permit ip host 1.1.1.1 any
 permit ip any host 1.1.1.1
!
policy-map NETFLOW-MAP
 class CMAP-JUST-R1
  netflow sampler High-SAMPLER
 class classs-default
  netflow sampler Med-SAMPLER
!
class-map CMAP-JUST-R1
 match access-group name ACL-JUST-R1
!
! These aren't really random, it does one then counts up to X then takes another
! 
flow-sampler-map High-SAMPLER
 mode random one-out-of 1
!
flow-sampler-map Med-SAMPLER
 mode random one-out-of 10

int <>
 service-policy output NETFLOW-MAP
```

### Another example
```
!
! Exporter
!
flow exporter MY-EXPORT
 destination 12.0.0.100
 transport udp 9996
!
! Flow Record
!
flow record MY-RECORD
 match ipv4 destination address
 match transport tcp destination-port
 match transpoct tcp source-port
 match ipv4 destination address
 match ipv4 source address
 match ipv4 fragmentation offset
 match ipv4 id
 collect counter packets
 collect counter bytes
!
! Flow Monitor
!
flow monitor MY-MONITOR
 exporter MY-EXPORT
 record MY-RECORD
 statistics packet size
 cache timeout inactive 21
!
int <>
 ip flow monitor MY-MONITOR input
```

### Verification commands

`show flow monitor <NAME>`

`show flow monitor <NAME> cache format record`

`show flow monitor <NAME> cache format table`

`show flow monitor <NAME> cache format csv`

`show flow monitor statistics`

`show flow interface`

`show flow exporter <NAME>`

`show flow exporter templates`

###### shows all the things you can match on and export

`show flow exporter export-ids netflow-v9`

###### show data towards the collector

`debug ip flow export`

`debug ip flow cache`