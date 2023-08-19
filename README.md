# one-node-ocp

Installation of a single node OpenShift Cluster.

## Link

* https://docs.openshift.com/container-platform/4.12/installing/installing_sno/install-sno-installing-sno.html
* https://www.adminsub.net/ipv4-subnet-calculator/10.0.4.0/24
* https://www.youtube.com/watch?v=syzwLwE3Xq4


## What do yo need?

Or what I used in this example.

* ESXi Host
* opnsense Firewall
* bind9 DNS Server

Had to use a bind9 server, so the domain and hostnames
are correclty resolved. And there are wildcard names.

The config looks like

```
ocp1            IN      A       10.0.4.221
*.ocp1          IN      A       10.0.4.221
```

* In opnsense add a static lease with the MAC address and IP
* Configure a VM with 32GB Ram, and 200GB Disk. Also
be sure you overwrite the MAC address, so you can link
the MAC address and IP address on your DHCP Server. 
* Setup a RHEL System. Here it's a rocky 9 system
* Login to the rocky system

```
$ OCP_VERSION=latest-4.12
$ ARCH=x86_64 
$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-client-linux.tar.gz -o oc.tar.gz
$ tar zxf oc.tar.gz
$ chmod +x oc
$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-install-linux.tar.gz -o openshift-install-linux.tar.gz
$ tar zxvf openshift-install-linux.tar.gz
$ chmod +x openshift-install
$ ISO_URL=$(./openshift-install coreos print-stream-json | grep location | grep $ARCH | grep iso | cut -d\" -f4)
$ curl -L $ISO_URL -o rhcos-live.iso
```

* Create `install-config.yaml`

```
$ mkdir ocp
$ cp install-config.yaml ocp
$ ./openshift-install --dir=ocp create single-node-ignition-config
$ alias coreos-installer='podman run --privileged --pull always --rm \
        -v /dev:/dev -v /run/udev:/run/udev -v $PWD:/data \
        -w /data quay.io/coreos/coreos-installer:release'
$ coreos-installer iso ignition embed -fi ocp/bootstrap-in-place-for-live-iso.ign rhcos-live.iso
```

* Upload `rhcos-live.iso` to the ESXI host, and use this iso for the new VM.

Debug the progress

    ./openshift-install --dir=ocp wait-for install-complete --log-level debug

Login the the OCP host

    ssh core@ocp1.internal.schlaepfer.com journalctl -b -f -u kubelet.service