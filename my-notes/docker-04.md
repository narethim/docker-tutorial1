# Linux Network Namespace Introduction

In this tutorial, we will learn what is Linux network namespace and how to use it.

Docker uses many Linux namespace technologies for isolation, there are user namespace, process namespace, etc. For network isolation docker uses Linux network namespace technology, each docker container has its own network namespace, which means it has its own IP address, routing table, etc.

First, let’s see how to create and check a network namespace. The lab environment we used today is a docker host which is created by docker-machine tool on Amazon AWS.

## Create and List Network Namespace

Use `ip netns add <network namespace name>` to create a network namespace, and `ip netns list` to list all network namepaces on the host.

```sh
[vagrant@docker-host hello-world]$ sudo ip netns add test1
[vagrant@docker-host hello-world]$ ip netns list
test1
```

## Delete Network Namespace

Use `ip netns delete <network namespace name>` to delete a network namespace.

```sh
[vagrant@docker-host hello-world]$ sudo ip netns delete test1
[vagrant@docker-host hello-world]$ ip netns list
[vagrant@docker-host hello-world]$ 
```

## Execute CMD within Network Namespace

How to check interfaces in a particular network namespace, we can use command `ip netns exec <network namespace name> <command>` like:

ubuntu@docker-host-aws:~$ sudo ip netns exec test1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
ubuntu@docker-host-aws:~$
ip a will list all ip interfaces within this test1 network namespaces. From the output we can see that the lo inteface is DOWN, we can run a command to let it up.

```sh
[vagrant@docker-host hello-world]$ sudo ip netns add test1

[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

`ip a` will list all ip interfaces within this `test1` network namespaces. From the output we can see that the `lo` inteface is `DOWN`, we can run a command to let it up.

```sh
sudo ip netns exec test1 ip link
sudo ip netns exec test1 ip link set dev lo up
sudo ip netns exec test1 ip link
```

Output:

```sh
[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
[vagrant@docker-host hello-world]$ 
[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip link set dev lo up
[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

The status of lo became UNKNOWN, please ignore that and go on.

## Add Interface to a Network Namespace

We will create a virtual interface pair, it has two virtual interfaces which are connected by a virtual cable

```sh
sudo ip link add veth-a type veth peer name veth-b
ip link
```

Output:

```sh
[vagrant@docker-host hello-world]$ sudo ip link add veth-a type veth peer name veth-b
[vagrant@docker-host hello-world]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:0d:96:53 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:c5:4a:26:52 brd ff:ff:ff:ff:ff:ff
10: veth8a3ecaf@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 72:80:d7:94:cb:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
13: veth-b@veth-a: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 72:cb:f8:15:8a:00 brd ff:ff:ff:ff:ff:ff
14: veth-a@veth-b: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 72:e4:36:73:27:42 brd ff:ff:ff:ff:ff:ff
```

All these two interfaces are located on localhost default network namespace. what we will do is move one of them to test1 network namespace, we can do this through:

```sh
sudo ip link set veth-b netns test1
ip link

sudo ip netns exec test1 ip link
```

Output:

```sh
[vagrant@docker-host hello-world]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:0d:96:53 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:c5:4a:26:52 brd ff:ff:ff:ff:ff:ff
10: veth8a3ecaf@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 72:80:d7:94:cb:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
14: veth-a@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 72:e4:36:73:27:42 brd ff:ff:ff:ff:ff:ff link-netnsid 1

[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
13: veth-b@if14: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 72:cb:f8:15:8a:00 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

Now, the interface `veth-b` is in network namespace `test1`.

## Assign IP address to `veth` interface

In the localhost to set `veth-a`

```sh
sudo ip addr add 192.168.1.1/24 dev veth-a
sudo ip link set veth-a up
ip link
```

Output:

```sh
[vagrant@docker-host hello-world]$ sudo ip addr add 192.168.1.1/24 dev veth-a
[vagrant@docker-host hello-world]$ sudo ip link set veth-a up
[vagrant@docker-host hello-world]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:0d:96:53 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:c5:4a:26:52 brd ff:ff:ff:ff:ff:ff
10: veth8a3ecaf@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 72:80:d7:94:cb:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
14: veth-a@if13: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether 72:e4:36:73:27:42 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

`veth-a` has an IP address, but its status is DOWN.

Now let’s set `veth-b` in `test1`.

```sh
sudo ip netns exec test1 ip addr add 192.168.1.2/24 dev veth-b
sudo ip netns exec test1 ip link set dev veth-b up
ip link
```

Output:

```sh
[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip addr add 192.168.1.2/24 dev veth-b
[vagrant@docker-host hello-world]$ sudo ip netns exec test1 ip link set dev veth-b up
[vagrant@docker-host hello-world]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:0d:96:53 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:c5:4a:26:52 brd ff:ff:ff:ff:ff:ff
10: veth8a3ecaf@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 72:80:d7:94:cb:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
14: veth-a@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 72:e4:36:73:27:42 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

After configured `veth-b` and up it, both `veth-a` and `veth-b` are UP. 

Now we can use `ping` to check their connectivity.

```sh
ping -c4 192.168.1.2
```

Output:

```sh
[vagrant@docker-host hello-world]$ ping -c4 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.078 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.071 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.079 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=0.081 ms

--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 0.071/0.077/0.081/0.007 ms
```

Please go to [Linux Switching – Interconnecting Namespaces](http://www.opencloudblog.com/?p=66) to learn more.
