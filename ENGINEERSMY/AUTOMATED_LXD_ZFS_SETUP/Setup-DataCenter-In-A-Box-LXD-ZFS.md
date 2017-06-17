# Setup DataCenter In a Box Using LXD + ZFS

## TL;DR

A simple method to setup a clustered environment mimicking your production system without needing to spend a dime in cloud computing cost in Azure.  Setup a full multi-VNet with 3 subnets in which to deploy 3 independent Consul node to form a fully running Consul cluster.

### Getting Started
Assumes on one of the below environments running Ubuntu 16.04:
- Xhyve (OSX) 
- Hyper-V (Windows)
- VirtualBox (no native Hypervisors like above)
- Azure (simulate full cluster using only one node; with fast network)

Get the script, make executable and run the script as found in the Nomad Box [repo](https://github.com/leowmjw/nomad-box).

The following is just explanation what the script is doing...

### Installing LXD + ZFS
The packages from the standard Ubuntu 16.04 is just too old so there is a need to get the latest packages (which has the important features like emulating VNet-like network + subnetworks):
```
# LXD needs the bleeding edge; thus add as per below to get at LXD + ZFS
export DEBIAN_FRONTEND=noninteractive && \
    apt-get install -y software-properties-common python-software-properties && \
    add-apt-repository ppa:ubuntu-lxc/lxd-git-master && \
    apt-get update && apt-get install -y lxd zfsutils-linux

```
To confirm the installation for LXD + ZFS is done correctly and using the latest version:
```
$ lxc --version
2.14

$ sudo zpool list
no pools available
```

### Setup LXD + ZFS

#### ZFS Setup ..
NOTE2: For a standalone VirtualBox, Xhyve or Hyper-V; the below would be the better option
```
$ sudo df -TH
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.7G     0  3.7G   0% /dev
tmpfs          tmpfs     731M   18M  713M   3% /run
/dev/sda1      ext4       32G  2.7G   29G   9% /
tmpfs          tmpfs     3.7G     0  3.7G   0% /dev/shm
tmpfs          tmpfs     5.3M     0  5.3M   0% /run/lock
tmpfs          tmpfs     3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/sda15     vfat      110M  3.6M  106M   4% /boot/efi
/dev/sdb1      ext4       15G   42M   14G   1% /mnt
tmpfs          tmpfs     731M     0  731M   0% /run/user/1000

$ free
              total        used        free      shared  buff/cache   available
Mem:        7131864      341656     5112716       17364     1677492     6483948
Swap:             0           0           0
```

As can be seen above, the root device (/dev/sda1), there is 29G available space.  The **ephemeral** device is currently not being used as sawp (see the `free` command output) so can be used as a ZFS cache.

On an Azure machine; 
```
# Code:
#     dd if=/dev/zero of=/var/lib/lxd/zfs-azure.img bs=1k count=1 seek=100M && \
#          zpool create my-zfs-pool /var/lib/lxd/zfs-azure.img && \
#          umount /mnt && zpool add -f my-zfs-pool cache /dev/sdb
$ AZURE_MODE="x" sudo ./test.bash 
1+0 records in
1+0 records out
1024 bytes (1.0 kB, 1.0 KiB) copied, 9.9e-05 s, 10.3 MB/s

$ df -TH
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.7G     0  3.7G   0% /dev
tmpfs          tmpfs     731M   18M  713M   3% /run
/dev/sda1      ext4       32G  2.7G   29G   9% /
tmpfs          tmpfs     3.7G     0  3.7G   0% /dev/shm
tmpfs          tmpfs     5.3M     0  5.3M   0% /run/lock
tmpfs          tmpfs     3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/sda15     vfat      110M  3.6M  106M   4% /boot/efi
/dev/sdb1      ext4       15G   42M   14G   1% /mnt
tmpfs          tmpfs     731M     0  731M   0% /run/user/1000
my-zfs-pool    zfs       104G     0  104G   0% /my-zfs-pool
```

The previous device will start losing data if it gets corrupted so is not advisable for production; at the minimum there should be a mirror. 

So if we wanted to do a mirror, it can be mirrored with the smaller device for 10GB let's say, like so:
```
dd if=/dev/zero of=/var/lib/lxd/zfs-mirror1.img bs=1k count=1 seek=10M & \
    dd if=/dev/zero of=/var/lib/lxd/zfs-mirror2.img bs=1k count=1 seek=10M && \
    zpool create my-zfs-pool mirror /var/lib/lxd/zfs-mirror1.img /var/lib/lxd/zfs-mirror2.img 
```

Once the underlying pool is setup; you can confirm its sturcture and even run `zpool scrub` to check the underlying setup has no errors.

```
$ sudo zpool status
  pool: my-zfs-pool
 state: ONLINE
  scan: none requested
config:

	NAME                              STATE     READ WRITE CKSUM
	my-zfs-pool                       ONLINE       0     0     0
	  mirror-0                        ONLINE       0     0     0
	    /var/lib/lxd/zfs-mirror1.img  ONLINE       0     0     0
	    /var/lib/lxd/zfs-mirror2.img  ONLINE       0     0     0

errors: No known data errors
```

For even better performance and redundancies; go with RaidZ setup; will be left as an exercise for the reader (Quiz time!)


#### LXD Headless Setup

Now that the underlying storage pool is ready, a headless initialization of LXD can be done by executing the below:
```
cat <<EOF | lxd init --verbose --preseed
# Daemon settings
config:
  core.https_address: 0.0.0.0:9999
  core.trust_password: passw0rd
  images.auto_update_interval: 36

# Storage pools
storage_pools:
- name: data
  driver: zfs
  config:
    source: my-zfs-pool/my-zfs-dataset

# Network devices
networks:
- name: lxd-my-bridge
  type: bridge
  config:
    ipv4.address: auto
    ipv6.address: none

# Profiles
profiles:
- name: default
  devices:
    root:
      path: /
      pool: data
      type: disk
- name: test-profile
  description: "Test profile"
  config:
    limits.memory: 2GB
  devices:
    test0:
      name: test0
      nictype: bridged
      parent: lxd-my-bridge
      type: nic
EOF

```

Finally the needed 3 subnets can be created:
```
$ lxc network create fsubnet1 ipv6.address=none ipv4.address=10.1.1.1/24 ipv4.nat=true
Network fsubnet1 created

$ lxc network create fsubnet2 ipv6.address=none ipv4.address=10.1.2.1/24 ipv4.nat=true
Network fsubnet2 created

$ lxc network create fsubnet3 ipv6.address=none ipv4.address=10.1.3.1/24 ipv4.nat=true
Network fsubnet3 created

$ lxc network list
+---------------+----------+---------+-------------+---------+
|     NAME      |   TYPE   | MANAGED | DESCRIPTION | USED BY |
+---------------+----------+---------+-------------+---------+
| docker0       | bridge   | NO      |             | 0       |
+---------------+----------+---------+-------------+---------+
| eth0          | physical | NO      |             | 0       |
+---------------+----------+---------+-------------+---------+
| fsubnet1      | bridge   | YES     |             | 0       |
+---------------+----------+---------+-------------+---------+
| fsubnet2      | bridge   | YES     |             | 0       |
+---------------+----------+---------+-------------+---------+
| fsubnet3      | bridge   | YES     |             | 0       |
+---------------+----------+---------+-------------+---------+
| lxd-my-bridge | bridge   | YES     |             | 0       |
+---------------+----------+---------+-------------+---------+

```
### Auto Install Consul Cluster

```
$ lxc remote list

+-----------------+------------------------------------------+---------------+--------+--------+
|      NAME       |                   URL                    |   PROTOCOL    | PUBLIC | STATIC |
+-----------------+------------------------------------------+---------------+--------+--------+
| images          | https://images.linuxcontainers.org       | simplestreams | YES    | NO     |
+-----------------+------------------------------------------+---------------+--------+--------+
| local (default) | unix://                                  | lxd           | NO     | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+
| ubuntu          | https://cloud-images.ubuntu.com/releases | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+
| ubuntu-daily    | https://cloud-images.ubuntu.com/daily    | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+
```

We'll get the latest Ubuntu images from the remote ubuntu-daily to live on the leading edge.  Images that come from the cloud-images repo has one important characteristic; it has `cloud-init` which we'll need to bootstrap the Consul nodes into the cluster.

Image can be obtained by launching a new image; and aliasing the repo:
```
$ lxc launch ubuntu-daily:17.04
Creating certain-krill
                                            
The container you are starting doesn't have any network attached to it.
  To create a new network, use: lxc network create
  To attach a network to a container, use: lxc network attach
$ lxc image alias create zesty b334ae64a8c3
$ lxc image list
+-------+--------------+--------+---------------------------------------+--------+----------+------------------------------+
| ALIAS | FINGERPRINT  | PUBLIC |              DESCRIPTION              |  ARCH  |   SIZE   |         UPLOAD DATE          |
+-------+--------------+--------+---------------------------------------+--------+----------+------------------------------+
| zesty | b334ae64a8c3 | no     | ubuntu 17.04 amd64 (daily) (20170617) | x86_64 | 158.53MB | Jun 17, 2017 at 3:37pm (UTC) |
+-------+--------------+--------+---------------------------------------+--------+----------+------------------------------+
$ lxc image info zesty
Fingerprint: b334ae64a8c3a6f873509a85efdcf2786e3b7fb4c4b20869327bbbf586042bfe
Size: 158.53MB
Architecture: x86_64
Public: no
Timestamps:
    Created: 2017/06/17 00:00 UTC
    Uploaded: 2017/06/17 15:37 UTC
    Expires: 2018/01/25 00:00 UTC
    Last used: 2017/06/17 15:37 UTC
Properties:
    serial: 20170617
    version: 17.04
    architecture: amd64
    description: ubuntu 17.04 amd64 (daily) (20170617)
    label: daily
    os: ubuntu
    release: zesty
Aliases:
    - zesty (zesty)
Auto update: enabled
Source:
    Server: https://cloud-images.ubuntu.com/daily
    Protocol: simplestreams
    Alias: 17.04
```
#### Executing cloud-init templates
HOWTO ..

### Internals

#### Setting up ZFS with proper RaidZ


