# auto-kube-dev

Automate the creation of a Kubernetes development environment.

It'll do somethings that you just don't wanna do ad-nauseum -- like install golang (and set your gopath), docker, and clone Kubernetes into the right spot, and have you set you can start hackin' instead of yak shaving (sorry, you'll still yak shave, too).

Assumes a CentOS 7 machine.

## Quick start

1. Copy the `./inventory/samples/example.inventory` to your own.
2. Run the `auto-kube-dev.yaml` playbook
3. ????
4. Profit.

Here's a sample run of the playbook:

```
$ ansible-playbook -i inventory/samples/example.inventory auto-kube-dev.yaml 
```

Or if you don't want to check the hardware requirements, go ahead and skip that like so:

```
$ ansible-playbook -i inventory/samples/example.inventory auto-kube-dev.yaml  -e 'perform_hardware_checks=false'
```

## Cloning the right repo

By default, we'll clone Kubernetes at master HEAD, but... There's a good chance you'll want to clone your own fork, or clone at a different branch or tag. So, you can specify that, too.

```
$ ansible-playbook -i inventory/samples/example.inventory auto-kube-dev.yaml -e 'kubernetes_version=v1.8.1' -e 'kubernetes_repo_url=https://github.com/kubernetes/kubernetes.git'
```

## Verifying this playbook got it right.

At very least, it should be able to run a `./hack/verify-bazel.sh`. (It should run, but, the success of which depends on the health of the branch you have checked out, try the `v1.8.1` tag for a good chance of it completing)

```
[centos@urbuildhost kubernetes]$ pwd
/home/centos/gocode/src/k8s.io/kubernetes
[centos@urbuildhost kubernetes]$ ./hack/verify-bazel.sh 
[centos@urbuildhost kubernetes]$ echo $?
0
```

## Requirements

Recommended: 24 gigs of RAM, and 30 gigs HDD space free in the root.

This playbook assumes a CentOS 7.

In Doug's case, he spins up VMs for this environment [basically using this concept](http://giovannitorres.me/create-a-linux-lab-on-kvm-using-cloud-images.html), and does something to resize the qcow2 image like:

```
[root@droctagon2 ~]# qemu-img resize /home/images/big.CentOS-7-x86_64-GenericCloud.qcow2 +60G
```
