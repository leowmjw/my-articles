# Setup DataCenter In a Box Using LXD + ZFS

## TL;DR

A simple method to setup a clustered environment mimicking your production system without needing to spend a dime in cloud computing cost in Azure.  Setup a full multi-VNet with 3 subnets in which to deploy 3 independent Consul node to form a fully running Consul cluster.

### Getting Started
Assumes on one of the below environments running Ubuntu 16.04:
- Xhyve (OSX) 
- Hyper-V (Windows)
- VirtualBox (no native Hypervisors like above)
- Azure (simulate full cluster using only one node; with fast network)

Get the script, make executable and run the script as found in the Nomad Box [repo](https://github.com/leowmjw/nomad-box). The main script inside the repo is in `/lxd/scripts/initlxd.sh`.  

The following is just explanation what the script is doing...

### Installing LXD + ZFS
The packages from the standard Ubuntu 16.04 is just too old so there is a need to get the latest packages (which has the important features like emulating VNet-like network + sub-networks):
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

The setup is automated by executing the `/tmp/script/initlxd.sh` script.
Below is the in-depth explanation of the why and how the script is structured.

#### ZFS Setup
For a development type setup, it is fine to use a file loopback as the backing for the ZFS setup.  In real production, as fast SSD partition and device would be advisable.

For a standalone VirtualBox, Xhyve or Hyper-V installation; the below would be the code to prepare the ZFS device to be made into a ZPool:
```
$ # On a VBox, the only valid assumption is that it will have a /var/lib/lxd/ directory after the installation of LXD
$ sudo dd if=/dev/zero of=/var/lib/lxd/zfs.img bs=1k count=1 seek=100M && \
        zpool create my-zfs-pool /var/lib/lxd/zfs.img

```
On an Azure machine; the standard `Standard_DS2_v2_Promo` instance looks like below:

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

As can be seen above, the root device (/dev/sda1), there is 29G available space.  The **ephemeral** device is currently not being used as swap (see the `free` command output) so can be used as a ZFS cache.

The code to setup the Azure node is as below:
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

Now with the underlying network setup, the goal is to auto boot up nodes and form a Consul Cluster automatically on an Ubuntu 17.04 installation with the latest Consul 0.8.4:

The available image repos:
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

Image can be obtained by copying image from remote; giving it an alias:
```
# Pull 17.04 and latest; have options to select ..
$ lxc image copy ubuntu-daily:17.04 local: --alias=zesty
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
Now that the images are available, there is a need to create a profile for the Foundation type nodes which will execute the cloud-init script via the user-data parameter.  

The cloud-init script will be similar to the ones used in the main Nomad Box Azure setup.  The lxd-foundation-init.sh used below can be found in the Nomad Box repo under `/lxd/scripts/lxd-foundation-init.sh`
```
# Attach profile for Foundation-type nodes
lxc profile create foundation
# Need to provide the cloud-init.sh scripts ..
lxc profile set foundation user.user-data - < /tmp/script/lxd-foundation-init.sh

```
Once the correct profile is setup as per above, the nodes can be initialized and started:
```
# Exec in and confirm it is running
lxc init zesty -p default -p foundation f1 && \
    lxc network attach fsubnet1 f1 eth0 && \
    lxc config device set f1 eth0 ipv4.address 10.1.1.4

lxc init zesty -p default -p foundation f2 && \
    lxc network attach fsubnet2 f2 eth0 && \
	lxc config device set f2 eth0 ipv4.address 10.1.2.4

lxc init zesty -p default -p foundation f3 && \
    lxc network attach fsubnet3 f3 eth0 && \
	lxc config device set f3 eth0 ipv4.address 10.1.3.4

lxc start f1 && lxc start f2 && lxc start f3

```

NOTE: Consul servers need to have custom node-id inside LXD, otherwise it gets confused that there are multiple Consul servers that are identical.  

To solve this, the Consul servers need to be executed with the flag `-disable-host-node-id` as documented [here](https://www.consul.io/docs/agent/options.html#_disable_host_node_id)

```
root@f3:/opt/consul# echo ${CONSUL_BIND_ADDRESS}
10.1.3.4
root@f3:/opt/consul# /opt/consul/consul agent -server -ui -bootstrap-expect=3 -data-dir=/tmp/consul   -config-dir=/opt/consul/consul.d -retry-join=10.1.1.4 -retry-join=10.1.2.4 -retry-join=10.1.3.4 -bind=${CONSUL_BIND_ADDRESS} -disable-host-node-id

```

### Output

For VirtualBox, you'll need to use sshuttle to access the LXD nodes network space of `10.1.0.0/16` with a command like below:
```
# Replace <PUBLIC_IP_BASTION>; for Azure it might be 52.187.116.82, VBox might be 192.168.1.132 etc
$ sshuttle -vH -r testadmin@<PUBLIC_IP_BASTION> 10.1.0.0/16

```

When you go to the IP address or node-name (f1,f2,f3) of any of the LXD nodes (http://10.1.1.4:8500 [http://f1:8500] or http://10.1.2.4:8500 [http://f2:8500] or http://10.1.3.4:8500 [http://f3:8500] )with Consul server and at Consul port, you will see the following:

   ![Consul Admin Dashboard](./IMAGES/Consul-Admin.png)

Other commands can be executed against any of the Consul servers to ensure the quorum is there:
```
testadmin@acme-nomad-dev-experiment-node-1:~$ CONSUL_HTTP_ADDR=http://10.1.3.4:8500 /opt/consul/consul members
Node  Address        Status  Type    Build  Protocol  DC
f1    10.1.1.4:8301  alive   server  0.8.4  2         dc1
f2    10.1.2.4:8301  alive   server  0.8.4  2         dc1
f3    10.1.3.4:8301  alive   server  0.8.4  2         dc1
```

Now, you have a usable Consul Server Cluster, next is to add more nodes that will do the load balancer(Director Nodes) and execute workloads (Worker Node).  That will be in the next article ..