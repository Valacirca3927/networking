On Cisco IOS `bgp soft-reconfig-backup` tells the router "if you must, save a entire table" otherwise rely on [RFC2918](https://www.rfc-editor.org/rfc/rfc2918), which is *dynamic updates*.

Soft reconfig is ancient, pre-RFC.

#### Soft Reconfig via Route Refresh (trusting the other device)!

`clear ip bgp <neighbor_ip> soft in`[^clear-soft]

[^clear-soft]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_bgp/configuration/xe-16/irg-xe-16-book/bgp-4-soft-configuration.html
