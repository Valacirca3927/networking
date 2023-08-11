These are mostly notes on Cisco IOS NetFlow Version 9

#### Export packet
```
  +---------------+------------------+--------------+--------------+---------+------------------+--------------+
  | Packet Header | Template FlowSet | Data FlowSet | Data FlowSet | ....... | Template Flowset | Data FlowSet |
  +---------------+------------------+--------------+--------------+---------+------------------+--------------+
```
[Cisco IOS NetFlow Version 9 Flow-Record Format - White Paper](https://www.cisco.com/en/US/technologies/tk648/tk362/technologies_white_paper09186a00800a3db9.pdf)

#### Terminology

* **Export packet** - Built by a device (for example, a router) with NetFlow services enabled, this type of packet is
addressed to another device (for example, a NetFlow collector). This other device processes the packet (parses, 
aggregates, and stores information on IP flows).

* **Export packet** - Built by a device (for example, a router) with NetFlow services enabled, this type of packet is
addressed to another device (for example, a NetFlow collector). This other device processes the packet (parses, 
aggregates, and stores information on IP flows).

* **Packet header** - The first part of an export packet, the packet header provides basic information about the packet,
such as the NetFlow version, number of records contained within the packet, and sequence numbering, enabling lost
packets to be detected.

* **FlowSet** - Following the packet header, an export packet contains information that must be parsed and interpreted
by the collector device. A FlowSet is a generic term for a collection of records that follow the packet header
in an export packet. There are two different types of FlowSets: template and data. An export packet contains
one or more FlowSets, and both template and data FlowSets can be mixed within the same export packet.

* **Template FlowSet** - A template FlowSet is a collection of one or more template records that have been grouped together
in an export packet.

* **Template record** - A template record is used to define the format of subsequent data records that may be received
in current or future export packets. It is important to note that a template record within an export packet does not
necessarily indicate the format of data records within that same packet. A collector application must cache any template 
records received, and then parse any data records it encounters by locating the appropriate template record within the cache.

* **Template ID** - The template ID is a unique number that distinguishes this template record from all other template records
produced by the same export device. A collector application that is receiving export packets from several devices should
be aware that uniqueness is not guaranteed across export devices. Thus, the collector should also cache the address of 
the export device that produced the template ID in order to enforce uniqueness.

* **Data FlowSet** - A data FlowSet is a collection of one or more data records that have been grouped together in an export packet.

* **Data record** - A data record provides information about an IP flow that exists on the device that produced an export packet.
Each group of data records (that is, each data FlowSet) references a previously transmitted template ID, which can be used
to parse the data contained within the records.

* **Options template** - An options template is a special type of template record used to communicate the format of data related
to the NetFlow process.

* **Options data record** - The options data record is a special type of data record (based on an options template)
with a reserved template ID that provides information about the NetFlow process itself.

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