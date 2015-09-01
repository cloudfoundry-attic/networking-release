# CF-BOSH Release containing several jobs to configure VM networking

This [CF-BOSH](http://docs.cloudfoundry.org/bosh/) release contains several jobs to help you configure VMs with
special networking properties:

* [Gateway](https://github.com/cf-platform-eng/networking-boshrelease/tree/master/jobs/gateway): allows to add a [default gateway](http://en.wikipedia.org/wiki/Default_gateway) to your vms
* [NAT](https://github.com/cf-platform-eng/networking-boshrelease/tree/master/jobs/nat): allows to create a [NAT](http://en.wikipedia.org/wiki/Network_address_translation) vm using [iptables](http://en.wikipedia.org/wiki/Iptables)
* [Routes](https://github.com/cf-platform-eng/networking-boshrelease/tree/master/jobs/routes): allows to add [IP routes](http://en.wikipedia.org/wiki/Routing_table) to your vms

## Usage

### Upload the BOSH release

To use this bosh release, first upload it to your bosh:

``` shell
bosh target BOSH_HOST
git clone https://github.com/cf-platform-eng/networking-boshrelease.git
cd networking-boshrelease
bosh upload release releases/networking/networking-2.yml
```

### Add the release to your BOSH deployment manifest

Add the appropriate release jobs and properties to your BOSH deployment manifest:

``` yaml
---
releases:
  - name: cf
    version: latest
  - name: networking
    version: latest

...

jobs:
  - name: haproxy
    templates:
      - name: nat
        release: networking
      - name: haproxy
        release: cf
    instances: 1
    resource_pool: default
    networks:
      - name: default
        default: [dns, gateway]
      - name: public
        static_ips:
          - 1.2.3.4
  - name: router
    templates:
      - name: gateway
        release: networking
      - name: routes
        release: networking
      - name: gorouter
        release: cf

...

properties:
  networking:
    nat:
      in_interface: eth0
      out_interface: eth1
    gateway:
      default: 0.haproxy.default.cf.microbosh
    routes:
      - net: 192.168.1.0
        netmask: 255.255.255.224
        interface: eth0
```

### Deploy using the BOSH deployment manifest

Using the previous created deployment manifest, now we can deploy it:

``` shell
bosh deployment path/to/deployment.yml
bosh -n deploy
```

## Contributing

In the spirit of [free software](http://www.fsf.org/licensing/essays/free-sw.html), **everyone** is encouraged to help improve this project.

Here are some ways *you* can contribute:

* by using alpha, beta, and prerelease versions
* by reporting bugs
* by suggesting new features
* by writing or editing documentation
* by writing specifications
* by writing code (**no patch is too small**: fix typos, add comments, clean up inconsistent whitespace)
* by refactoring code
* by closing [issues](https://github.com/cf-platform-eng/networking-boshrelease/issues)
* by reviewing patches


### Submitting an Issue
We use the [GitHub issue tracker](https://github.com/cf-platform-eng/networking-boshrelease/issues) to track bugs and features.
Before submitting a bug report or feature request, check to make sure it hasn't already been submitted.
You can indicate support for an existing issue by voting it up.
When submitting a bug report, please include a [Gist](http://gist.github.com/) that includes a stack trace and any
details that may be necessary to reproduce the bug, including your gem version, Ruby version, and operating system.
Ideally, a bug report should include a pull request with failing specs.

### Submitting a Pull Request

1. Fork the project.
2. Create a topic branch.
3. Implement your feature or bug fix.
4. Commit and push your changes.
5. Submit a pull request.

### Create new release

#### Creating a final release

If you need to create a new final release, you will need to get read/write API credentials to the [@cloudfoundry-community](https://github.com/cloudfoundry-community) s3 account.

Please email [Dr Nic Williams](mailto:&#x64;&#x72;&#x6E;&#x69;&#x63;&#x77;&#x69;&#x6C;&#x6C;&#x69;&#x61;&#x6D;&#x73;&#x40;&#x67;&#x6D;&#x61;&#x69;&#x6C;&#x2E;&#x63;&#x6F;&#x6D;) and he will create unique API credentials for you.

Create a `config/private.yml` file with the following contents:

``` yaml
---
blobstore:
  s3:
    access_key_id:     ACCESS
    secret_access_key: PRIVATE
```

You can now create final releases for everyone to enjoy!

``` shell
bosh create release
# test this dev release
git commit -m "updated networking bosh release"
bosh create release --final
git commit -m "creating vXYZ release"
git tag vXYZ
git push origin master --tags
```

## Copyright

See [LICENSE](https://github.com/cf-platform-eng/networking-boshrelease/blob/master/LICENSE) for details.
Copyright (c) 2014 [Pivotal Software, Inc](http://www.pivotal.io/).

Based on the [Rakuten BOSH routing release](https://github.com/rakutentech/bosh-routing-release).
