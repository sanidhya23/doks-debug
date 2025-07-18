**Description**

A Docker image with Kubernetes manifests for investigation and troubleshooting your cluster.

**Purpose** 

This is forked from [DOKS](https://github.com/digitalocean/doks-debug). This image used to deploy a debug pod or to deploy ephemerel container within a pod to provide debugging utilities for troubleshooting

**Usage**

Run as pod
Deploy as demonset so that pod is deployed on all the available nodes
```
kubectl apply -f daemonset.yaml
```

Deploy as deployment so that pod(s) is deployed as per the node affinity or tolerations specified in the manifest file
```
kubectl apply -f deployment.yaml
```


This Pod will do following:
- Use hostPID, hostIPC, and hostNetwork.
- Mount the entire host filesystem to /host in the containers.
- Mount the containerd socket at /run/containerd/containerd.sock from the host into the container.
- In order to make use of these workloads, you can exec into a pod of choice by name:

```
kubectl -n kube-system exec -it my-pod-name -- bash
```

If pod is deployed as demonset, you can exec into the debug pod on the specific node with:
```
NODE_NAME="my-node-name"
POD_NAME=$(kubectl -n kube-system get pods --field-selector spec.nodeName=${NODE_NAME} -ojsonpath='{.items[0].metadata.name}')
kubectl -n kube-system exec -it ${POD_NAME} bash
```

Once you're in, you have access to the set of tools listed in the Dockerfile. This includes:
- [vim](https://github.com/vim/vim) - is a greatly improved version of the good old UNIX editor Vi.
- [screen](https://www.gnu.org/software/screen) - is a full-screen window manager that multiplexes a physical terminal between several processes, typically interactive shells.
- [curl](https://github.com/curl/curl) - is a command-line tool for transferring data specified with URL syntax.
- [jq](https://github.com/stedolan/jq) - is a lightweight and flexible command-line JSON processor.
- [dnsutils](https://packages.debian.org/stretch/dnsutils) - includes various client programs related to DNS that are derived from the BIND source tree, specifically [dig](https://linux.die.net/man/1/dig), [nslookup](https://linux.die.net/man/1/nslookup), and [nsupdate](https://linux.die.net/man/8/nsupdate).
- [iputils-ping](https://packages.debian.org/stretch/iputils-ping) - includes the [ping](https://linux.die.net/man/8/ping) tool that sends ICMP ECHO_REQUEST packets to a host in order to test if the host is reachable via the network.
- [tcpdump](https://www.tcpdump.org) - a powerful command-line packet analyzer; and libpcap, a portable C/C++ library for network traffic capture.
- [traceroute](https://linux.die.net/man/8/traceroute) - tracks the route packets taken from an IP network on their way to a given host.
- [net-tools](https://packages.debian.org/stretch/net-tools) - includes the important tools for controlling the network subsystem of the Linux kernel, specifically [arp](http://man7.org/linux/man-pages/man8/arp.8.html), [ifconfig](https://linux.die.net/man/8/ifconfig), and [netstat](https://linux.die.net/man/8/netstat).
- [netcat](https://linux.die.net/man/1/nc) - is a multi-tool for interacting with TCP and UDP; it can open TCP connections, send UDP packets, listen on arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6.
- [iproute2](https://wiki.linuxfoundation.org/networking/iproute2) - is a collection of utilities for controlling TCP / IP networking and traffic control in Linux.
- [strace](https://github.com/strace/strace) - is a diagnostic, debugging and instructional userspace utility with a traditional command-line interface for Linux. It is used to monitor and tamper with interactions between processes and the Linux kernel, which include system calls, signal deliveries, and changes of process state.
- [dstat](http://dag.wiee.rs/home-made/dstat) - is a versatile replacement for vmstat, iostat, netstat and ifstat. Dstat overcomes some of their limitations and adds some extra features, more counters and flexibility. Dstat is handy for monitoring systems during performance tuning tests, benchmarks or troubleshooting.
- [htop](https://hisham.hm/htop) - is interactive process viewer for Unix systems.
- [atop](https://www.atoptool.nl) - is an advanced interactive monitor for Linux-systems to view the load on system-level and process-level.
- [wget](https://www.gnu.org/software/wget) - for retrieving files using HTTP, HTTPS, FTP and FTPS.
- [crictl](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md) - A CLI for CRI endpoints. Configured to use /run/containerd/containerd.sock as a default endpoint.

**Examples**
Run crictl
```
crictl ps
crictl pods
```

Run chroot and systemctl
```
chroot /host /bin/bash
systemctl status kubelet
journalctl -xe
journalctl -u kubelet
```

Run as ephemeral container
Let us assume if you wish to debug a pod named test-nginx-7b646d5f9f-49rxc in namespace called test. Then you can spinup an ephemeral container within an existing pod or create it inside the copy of that pod

Create in existing pod
```
kubectl -n test debug test-nginx-7b646d5f9f-49rxc -it --container=mycontainer --image=<ECR Repo>/doks-debug:latest --profile general -- sh
```

Create a copy of the targeted pod with name my-copied-pod and create container within it
```
kubectl -n test debug test-nginx-7b646d5f9f-49rxc -it --copy-to=my-copied-pod --container=mycontainer --image=<ECR Repo>/doks-debug:latest --profile general -- sh
```
Exec into the pod by specifying the container name usig -c flag
```
kubectl -n kube-system exec -it test-nginx-7b646d5f9f-49rxc -c mycontainer -- bash
```
If pod has [shareProcessNamespace](https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/) spec enabled ```shareProcessNamespace: true```, then ephemeral container can attach to the process running in other conainers running within the pod

Examples
Strace or restart a process
```
htop  # shows all processes in all containers within pod
kill -HUP <pid>
strace -p <pid> -o /tmp/app.trace
```

Access container's file system via process
```
ls -l /proc/<pid>/root
```