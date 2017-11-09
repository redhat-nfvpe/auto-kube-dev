# auto-kube-dev

Automate the creation of a Kubernetes development environment

## Quick start

1. Copy the `./inventory/samples/example.inventory` to your own.
2. Run the `auto-kube-dev.yaml` playbook
3. ????
4. Profit.

Here's a sample run of the playbook:

```
ansible-playbook -i inventory/samples/example.inventory auto-kube-dev.yaml 
```

Or if you don't want to check the hardware requirements, go ahead and skip that like so:

```
ansible-playbook -i inventory/samples/example.inventory auto-kube-dev.yaml  -e 'perform_hardware_checks=false'
```

## Requirements

Recommended: 24 gigs of RAM, and 30 gigs HDD space free in the root.

In Doug's case, he spins up VMs for this environment, and does something like:

```
[root@droctagon2 ~]# qemu-img resize /home/images/big.CentOS-7-x86_64-GenericCloud.qcow2 +60G
```
