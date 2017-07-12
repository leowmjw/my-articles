#  Setup DataCenter In a Box Using LXD + ZFS

## Objective
- To document a method to deploy the so that can simulate a full VNet like in Azure Cloud system on your own laptop

## Audience
- Developers interested deploying Nomad Box without the needed for an Azure account
- Ops interested in automating a dev environment utilizing LXD, ZFS as a base for easily cloning development environments

## References
- Code can be found in the current [Nomad Box](https://www.github.com/leowmjw/nomad-box) code
- The main script inside the repo is in `/lxd/scripts/initlxd.sh`.
 
## Highlights 
- Learn what are the base hypervisor's needed to deploy this: VirtualBox, Xhyve, MS Hyper-V 
- Learn how to install all the pre-reqs needed for LXD, ZFS installation
- Learn how to prepare the ZFS Pool for use by ZFS
- Learn how to prepare a headless install and config of LXD; with some useful default profiles 
- Learn how to pull images that have cloud-init capablities and link their automation to with the profiles
- Test the automatic bootstrap of a consul cluster

## Content link
- [Getting Started](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#getting-started)
- [Installing LXD + ZFS](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#installing-lxd--zfs)
- [Setup LXD + ZFS](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#setup-lxd--zfs)
- [LXD Headless Setup](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#lxd-headless-setup)
- [Auto Install Consul Cluster](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#auto-install-consul-cluster)
- [Output](./Setup-DataCenter-In-A-Box-LXD-ZFS.md#output)