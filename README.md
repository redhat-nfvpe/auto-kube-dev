# auto-kube-dev

Automate the creation of a Kubernetes development environment.

It'll do somethings that you just don't wanna do ad-nauseam -- like install golang (and set your gopath), docker, and clone Kubernetes into the right spot. With that all set you can start hackin' instead of yak shaving (sorry, you'll still yak shave, too).

Assumes a CentOS 7 machine.

## Quick start

1. Copy the `./inventory/samples/example.inventory` to your own.
2. Run the `auto-kube-dev.yaml` playbook
3. ????
4. Profit.

## Example playbook run

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

This playbook assumes a CentOS 7 machine.

In Doug's case, he spins up VMs for this environment [basically using this concept](http://giovannitorres.me/create-a-linux-lab-on-kvm-using-cloud-images.html), and does something to resize the qcow2 image like:

```
[root@droctagon2 ~]# qemu-img resize /home/images/big.CentOS-7-x86_64-GenericCloud.qcow2 +60G
```

## A sample compilation & run of Kubernetes

Some example build instructions from [Tomo](https://github.com/s1061123)! First, build the binaries, Tomo's quick method:

```
env KUBE_FASTBUILD=true KUBE_VERBOSE=8  ./build/run.sh make
```

Start a tmux session.

```
tmux new -s kuberun
```

Run a local cluster in that tmux session (do this as root). Double check that the `$ip` variable works for you in your scenario.

```
[root@buildkube kubernetes]# ./hack/install-etcd.sh
[root@buildkube kubernetes]# export PATH=$(pwd)/third_party/etcd:${PATH} 
[root@buildkube kubernetes]# ip=$(ip a | grep 192.168.1. | awk '{print $2}' | sed -e 's|/24||')
[root@buildkube kubernetes]# LOG_LEVEL=4 API_HOST=$ip ENABLE_RBAC=true ./hack/local-up-cluster.sh -o _output/dockerized/bin/linux/amd64/
```

Exit that tmux session (`ctrl+b, d`).

Now... Try to use it. Note: Don't do this as root, it won't work.

```
[centos@buildkube kubernetes]$   export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
[centos@buildkube kubernetes]$   cluster/kubectl.sh get nodes
No resources found.
```

For fun, start a pod. Run a test using [the pod from this gist](https://gist.github.com/dougbtv/e407fb95fe19de05637e5be9a0e4b252)

```
[centos@buildkube kubernetes]$   cluster/kubectl.sh create -f ~/debug.yaml 
replicationcontroller "debugging" created
```

Watch that pod come up.

```
[centos@buildkube kubernetes]$ watch -n1 cluster/kubectl.sh get pods -o wide --all-namespaces
```

And you can see if that's doing anything...

```
[centos@buildkube kubernetes]$ cluster/kubectl.sh get pods
NAME              READY     STATUS    RESTARTS   AGE
debugging-m8c4p   1/1       Running   0          4m
[centos@buildkube kubernetes]$ cluster/kubectl.sh exec -it debugging-m8c4p -- ping -c2 4.2.2.2 
PING 4.2.2.2 (4.2.2.2) 56(84) bytes of data.
64 bytes from 4.2.2.2: icmp_seq=1 ttl=57 time=11.2 ms
64 bytes from 4.2.2.2: icmp_seq=2 ttl=57 time=10.5 ms
```

And there you have it!