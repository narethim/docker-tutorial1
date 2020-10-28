# Container Port Mapping in Bridge networking

Through Bridge Networking Deep Dive we know that by default Docker containers can make connections to the outside world, but the outside world cannot connect to containers. Each outgoing connection will appear to originate from one of the host machine’s own IP addresses thanks to an iptables masquerading rule on the host machine that the Docker server creates when it starts: [1]

```sh
sudo iptables -t nat -L -n
ifconfig docker0
```

Output:

```sh
[vagrant@docker-host ~]$ sudo iptables -t nat -L -n
...
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
...

[vagrant@docker-host ~]$ ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:c5ff:fe4a:2652  prefixlen 64  scopeid 0x20<link>
        ether 02:42:c5:4a:26:52  txqueuelen 0  (Ethernet)
        RX packets 8004  bytes 327177 (319.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8366  bytes 14687154 (14.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...
```

The Docker server creates a `masquerade` rule that let containers connect to IP addresses in the outside world.

## Docker0 bridge

## Bind Container port to the host

Start a nginx container which export port 80 and 443. we can access the port from inside of the docker host.

```sh
sudo docker run -d  --name demo nginx
sudo docker ps
sudo docker inspect --format {{.NetworkSettings.IPAddress}} demo
curl 172.17.0.2
```

Output:

```sh
[vagrant@docker-host ~]$ sudo docker run -d  --name demo nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
bb79b6b2107f: Pull complete 
111447d5894d: Pull complete 
a95689b8e6cb: Pull complete 
1a0022e444c2: Pull complete 
32b7488a3833: Pull complete 
Digest: sha256:ed7f815851b5299f616220a63edac69a4cc200e7f536a56e421988da82e44ed8
Status: Downloaded newer image for nginx:latest
a5985c9550241c2f9102a0927fd74fce9310dfecd46eb21b961b807c91ba1544
[vagrant@docker-host ~]$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
a5985c955024        nginx               "/docker-entrypoint.…"   1 second ago        Up Less than a second   80/tcp              demo
a453d75860c2        centos:7            "/bin/bash -c 'while…"   About an hour ago   Up About an hour                            test2
ae73a290c5b6        centos:7            "/bin/bash -c 'while…"   2 hours ago         Up 2 hours                                  test1
[vagrant@docker-host ~]$ sudo docker inspect --format {{.NetworkSettings.IPAddress}} demo
172.17.0.4
[vagrant@docker-host ~]$ curl 172.17.0.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

If we want to access the nginx web from outside of the docker host, we must bind the port to docker host like this:

```sh
sudo docker run -d  -p 80 --name demo nginx
sudo docker ps

curl 192.168.205.10:32768
ifconfig eth1
```

Output:

```sh
[vagrant@docker-host ~]$ sudo docker run -d  -p 80 --name demo nginx
85fa41611ab9e9bed3d969a243ce5955eff2f0204898c842389c510444664116
[vagrant@docker-host ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
85fa41611ab9        nginx               "/docker-entrypoint.…"   14 seconds ago      Up 13 seconds       0.0.0.0:32768->80/tcp   demo
[vagrant@docker-host ~]$ 
[vagrant@docker-host ~]$ curl 192.168.205.10:32768
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[vagrant@docker-host ~]$ ifconfig eth1
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.205.10  netmask 255.255.255.0  broadcast 192.168.205.255
        inet6 fe80::a00:27ff:fe0d:9653  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:0d:96:53  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 23  bytes 2684 (2.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

If we want to point out which port on host want to bind:

```sh
docker run -d  -p 80:80 --name demo1 nginx
docker ps

curl 192.168.205.10:80
ifconfig eth1
```

Output:

```sh
[vagrant@docker-host ~]$ docker run -d  -p 80:80 --name demo1 nginx
db494545c3d3bb8d99cf095571dfce9595670f9f17e9d720b98c37cbf6958516
[vagrant@docker-host ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
db494545c3d3        nginx               "/docker-entrypoint.…"   11 seconds ago      Up 10 seconds       0.0.0.0:80->80/tcp      demo1
85fa41611ab9        nginx               "/docker-entrypoint.…"   5 minutes ago       Up 5 minutes        0.0.0.0:32768->80/tcp   demo

[vagrant@docker-host ~]$ curl 192.168.205.10:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## What happened

It’s iptables

```sh
[vagrant@docker-host ~]$ sudo iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  172.17.0.2           172.17.0.2           tcp dpt:80
MASQUERADE  tcp  --  172.17.0.3           172.17.0.3           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:32768 to:172.17.0.2:80
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.3:80
```

```sh
[vagrant@docker-host ~]$ sudo iptables -t nat -nvxL
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination
       1       44 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 1 packets, 76 bytes)
    pkts      bytes target     prot opt in     out     source               destination
       3      180 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 3 packets, 196 bytes)
    pkts      bytes target     prot opt in     out     source               destination
      14      927 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0
       0        0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80
       0        0 MASQUERADE  tcp  --  *      *       172.17.0.3           172.17.0.3           tcp dpt:80

Chain DOCKER (2 references)
    pkts      bytes target     prot opt in     out     source               destination
       0        0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0
       1       60 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:32768 to:172.17.0.2:80
       2      120 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.3:80

```

## References

[1]	[https://docs.docker.com/engine/userguide/networking/default_network/binding/](https://docs.docker.com/engine/userguide/networking/default_network/binding/)
