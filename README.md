# CF-BOSH Release containing several jobs to configure VM networking

This [BOSH](http://bosh.io/) release contains several jobs to help you configure VMs with
special networking properties:

* [Gateway](jobs/gateway): allows to add a [default gateway](http://en.wikipedia.org/wiki/Default_gateway) to your vms
* [NAT](jobs/nat): allows to create a [NAT](http://en.wikipedia.org/wiki/Network_address_translation) vm using [iptables](http://en.wikipedia.org/wiki/Iptables)
* [Routes](jobs/routes): allows to add [IP routes](http://en.wikipedia.org/wiki/Routing_table) to your vms

## Usage

### Upload the BOSH release

To use this bosh release, first upload it to your bosh:

```bash
bosh upload release https://bosh.io/d/github.com/cloudfoundry/networking-release
```

### Add the release to your BOSH deployment manifest

Add the networking release jobs and properties to your BOSH deployment manifest:

```yaml
releases:
  - name: cf
    version: latest
  - name: networking                                # +
    version: latest                                 # +
...
instance_groups:
  - name: haproxy
    jobs:
      - name: nat                                   # +
        release: networking                         # +
        properties:                                 # +
          networking.nat:                           # +
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
  - name: router
    jobs:
      - name: gateway                               # +
        release: networking                         # +
        properties:                                 # +
          networking.gateway:                       # +
            default: 0.haproxy.default.cf.microbosh # +
      - name: routes                                # +
        release: networking                         # +
        properties:                                 # +
          networking.routes:                        # +
            - net: 192.168.1.0                      # +
              netmask: 255.255.255.224              # +
              interface: eth0                       # +
              gateway: 10.9.9.1                     # +
      - name: gorouter
        release: cf
```

### Deploy using the BOSH deployment manifest

Using the previous created deployment manifest, now we can deploy it:

``` shell
bosh deployment path/to/deployment.yml
bosh deploy
```

## References

Based on the [Rakuten BOSH routing release](https://github.com/rakutentech/bosh-routing-release).

## License

Apache License Version 2.0 - see [LICENSE](LICENSE) for details.
