A very basic setup, that assumes a working underlay. I implemented this on c7200s via GNS3 running `15.2(4)S7`. My underlay was IS-IS to router loopbacks.
 
#### Site 1 - xTR
##### config
```
R18# show run | s lisp
router lisp
 database-mapping 192.168.100.0/24 18.18.18.18 priority 1 weight 50
 ipv4 itr map-resolver 16.16.16.16
 ipv4 itr
 ipv4 etr map-server 16.16.16.16 key cisco
 ipv4 etr
 exit
```
##### verify 
```
R18# show ip lisp map-cache 
LISP IPv4 Mapping Cache for EID-table default (IID 0), 2 entries

0.0.0.0/0, uptime: 00:19:42, expires: never, via static send map-request
  Negative cache entry, action: send-map-request
192.168.101.0/24, uptime: 00:10:08, expires: 23:49:44, via map-reply, complete
  Locator      Uptime    State      Pri/Wgt
19.19.19.19  00:10:08  up           1/50 
```

#### Site 2 - xTR
##### config
```
R19# show run | s lisp
router lisp
 database-mapping 192.168.101.0/24 19.19.19.19 priority 1 weight 50
 ipv4 itr map-resolver 16.16.16.16
 ipv4 itr
 ipv4 etr map-server 16.16.16.16 key cisco
 ipv4 etr
 exit
```
##### verify
```
R19#show ip lisp map-cache 
LISP IPv4 Mapping Cache for EID-table default (IID 0), 2 entries

0.0.0.0/0, uptime: 00:11:50, expires: never, via static send map-request
  Negative cache entry, action: send-map-request
192.168.100.0/24, uptime: 00:11:29, expires: 23:48:23, via map-reply, complete
  Locator      Uptime    State      Pri/Wgt
  18.18.18.18  00:11:29  up           1/50
```
##### MS/MR
###### config
```
R16# show run | s lisp
router lisp
 site 1
  authentication-key cisco
  eid-prefix 192.168.100.0/24
  exit
 !
 site 2
  authentication-key cisco
  eid-prefix 192.168.101.0/24
  exit
 !
 ipv4 map-server
 ipv4 map-resolver
 exit
```
##### verify
```
R16# show lisp site name 1
Site name: 1
Allowed configured locators: any
Allowed EID-prefixes:
  EID-prefix: 192.168.100.0/24 
    First registered:     00:25:12
    Routing table tag:    0
    Origin:               Configuration
    Merge active:         No
    Proxy reply:          No
    TTL:                  1d00h
    State:                complete
    Registration errors:  
      Authentication failures:   0
      Allowed locators mismatch: 0
    ETR 10.0.0.23, last registered 00:00:28, no proxy-reply, no map-notify
                   TTL 1d00h, no merge, nonce 0x3E715231-0x150380FC
                   state complete
      Locator      Local  State      Pri/Wgt
      18.18.18.18  yes    up           1/50 

R16# show lisp site name 2
Site name: 2
Allowed configured locators: any
Allowed EID-prefixes:
  EID-prefix: 192.168.101.0/24 
    First registered:     00:25:24
    Routing table tag:    0
    Origin:               Configuration
    Merge active:         No
    Proxy reply:          No
    TTL:                  1d00h
    State:                complete
    Registration errors:  
      Authentication failures:   0
      Allowed locators mismatch: 0
    ETR 10.0.0.26, last registered 00:00:37, no proxy-reply, no map-notify
                   TTL 1d00h, no merge, nonce 0x2F281A3C-0x0760FD58
                   state complete
      Locator      Local  State      Pri/Wgt
      19.19.19.19  yes    up           1/50 
```
