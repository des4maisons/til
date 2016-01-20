
* When playing, it's common to tweak cloud-config requiring
    booting/rebooting/destroying many machines. Maybe get things working before
    trying to set up mac-mini? or make it the dedicated etcd instance?

* Network bootstrapping: if you have 1 etcd node, hard-code its ip (some
  providers allow you to use the variable `$private_ipv4`), and when
  you bring up the rest of the cluster, configure fleetctl and etcdctl to look
  to the machine at that IP.

    ```yaml
      etcd:
        addr: $private_ipv4:4001
        peer-addr: $private_ipv4:7001
    ```

* fleet is like systemd for a cluster.

* To group members of a core-service cluster together, machines must be told to
  register their peer-addr to a particular discovery token when they boot. They
  are given a list of other members of the core-cluster upon registration. equip
  members of an etcd-core-services cluster with a discovery token.
  coreos/etcd.io provides such a discovery service. You can generate a
  discovery token with this service here:
     https://discovery.etcd.io/new?size=X, X=size of service cluster.

    ```yaml
    coreos:
      etcd:
        discovery: https://discovery.etcd.io/<token>
        addr: $private_ipv4:4001
        peer-addr: $private_ipv4:7001
    ```

* Configure non-core nodes to connect to core-nodes by hardcoding ip address:

    ```yaml
    coreos:
      fleet:
        metadata: "role=worker,cabinet=two,disk=spinning"
        etcd_servers:
    "http://10.0.0.101:4001,http://10.0.0.102:4001,http://10.0.0.103:4001,http://10.0.0.104:4001,http://10.0.0.105:4001"
      locksmith:
        endpoint:
    "http://10.0.0.101:4001,http://10.0.0.102:4001,http://10.0.0.103:4001,http://10.0.0.104:4001,http://10.0.0.105:4001"
      units:
        - name: etcd.service
          mask: true
        - name: fleet.service
          command: start
    write_files:
      - path: /etc/profile.d/etcdctl.sh
        permissions: 0644
        owner: core
        content: |
          # configure etcdctl to work with our etcd servers set above
          export
    ETCDCTL_PEERS="http://10.0.0.101:4001,http://10.0.0.102:4001,http://10.0.0.103:4001,http://10.0.0.104:4001,http://10.0.0.105:4001"
      - path: /etc/profile.d/fleetctl.sh
        permissions: 0644
        owner: core
        content: |
          # configure fleetctl to work with our etcd servers set above
          export FLEETCTL_ENDPOINT=unix:///var/run/fleet.sock
          export FLEETCTL_EXPERIMENTAL_API=true
    ```

* In core-os, automatic updates happen by writing updates to a passive
    partition. On reboot, the active partition is swapped for the passive
    partition.

* The entries stored in the discovery service provided by etcd/coreos have a
  TTL of 7 days. It's possible for a machine to register to a service that no
  longer has any active nodes. The first node that registers puts the state of
  the cluster to "started", so that further leader election will happen between
  the cluster nodes. Thus, when a cluster goes stale, you need to discard the
  discovery url and create a new one.

* yaml validator: http://www.yamllint.com/ or `coreos-cloudinit -validate`

* To troubleshoot cloud bootstrapping (such as invalid cloud-config files), try
  this command: `journalctl _EXE=/usr/bin/coreos-cloudinit`

* flanneld allocates IP addresses to each *container* across a cluster. It
  allocates a subnet to each worker from which containers may be allocated IP
  addresses, and aggregates all the subnets into one big subnet. It uses etcd
  to maintain the subnet-machine mapping. I believe docker0 is the aggregated
  subnet. flanneld runs on each host so that every host knows to which machine
  it should send packets on docker0. Thus ???flanneld "uses packet encapsulation
  to create a virtual overlay network that spans the whole cluster"

* `manage_etc_hosts: localhost` tells the coreos to manage the localhost entry
  in /etc/hosts,  which is useful when there is no DNS infrastructure in place
  to resolve own hostname (eg when using vagrant)

* Tip: use attached storage for docker images, since debugging building of
  images can use up large amounts of storage. Mount external storage to
  `/var/lib/docker`.
