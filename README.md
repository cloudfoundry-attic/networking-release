# CF-BOSH Release containing several jobs to configure VM networking

This [BOSH](http://bosh.io/) release contains several jobs to help you configure VMs with
special networking properties:

* [Gateway](jobs/gateway): allows to add a [default gateway](http://en.wikipedia.org/wiki/Default_gateway) to your vms
* [NAT](jobs/nat): allows to create a [NAT](http://en.wikipedia.org/wiki/Network_address_translation) vm using [iptables](http://en.wikipedia.org/wiki/Iptables)
* [IPtables](jobs/iptables): allows to add custom iptables rules [iptables](http://en.wikipedia.org/wiki/Iptables)
* [Routes](jobs/routes): allows to add [IP routes](http://en.wikipedia.org/wiki/Routing_table) to your vms
* [MTU](jobs/set_mtu): allows to override [MTU](https://en.wikipedia.org/wiki/Maximum_transmission_unit) on your vms

## Usage

### Add the release to your BOSH deployment manifest

Add the networking release jobs and properties to your BOSH deployment manifest:

```yaml
releases:
  - name: cf
    version: latest
  - name: networking                                  # +
    version: latest                                   # +
...
instance_groups:
  - name: haproxy
    jobs:
      - name: nat                                     # +
        release: networking                           # +
        properties:                                   # +
          networking:                                 # +
            nat:                                      # +
              in_interface: eth0                      # +
              out_interface: eth1                     # +
      - name: haproxy
        release: cf
    networks:
      - name: default
        default: [dns, gateway]
      - name: public
        static_ips:
          - 1.2.3.4
  - name: natgateway
    jobs:
      - name: nat
        release: networking
        properties:
          networking:
            nat:
              out_interface: eth0
      - name: iptables
        release: networking
        properties:
          nat:
            POSTROUTING:
            - -o eth0 -j MASQUERADE
  - name: router
    jobs:
      - name: gateway                                 # +
        release: networking                           # +
        properties:                                   # +
          networking:                                 # +
            gateway:                                  # +
              default: 0.haproxy.default.cf.microbosh # +
      - name: routes                                  # +
        release: networking                           # +
        properties:                                   # +
          networking.routes:                          # +
            - net: 192.168.1.0                        # +
              netmask: 255.255.255.224                # +
              interface: eth0                         # +
              gateway: 10.9.9.1                       # +
      - name: port_forwarding                         # +
        release: networking                           # +
        properties:                                   # +
          networking:
            port_forwarding:                          # +
              - external_port: 9200                   # +
                internal_ip: 1.2.3.10                 # +
                internal_port: 9200                   # +
              - external_port: 9292                   # +
                internal_ip: 1.2.3.11                 # +
                internal_port: 9292                   # +
      - name: gorouter
        release: cf
```

## References

Based on the [Rakuten BOSH routing release](https://github.com/rakutentech/bosh-routing-release).

## License

Apache License Version 2.0 - see [LICENSE](LICENSE) for details.
