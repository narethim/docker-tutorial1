# Docker Command Line Step by Step

* [Docker Command Line Step by Step](https://docker-k8s-lab.readthedocs.io/en/latest/docker/docker-cli.html)

## 1. Docker Images

### docker pull

```sh
docker pull ubuntu:14.04
```

Output:

```sh
[vagrant@docker-host ~]$ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
2e6e20c8e2e6: Pull complete 
95201152d9ff: Pull complete 
5f63a3b65493: Pull complete 
Digest: sha256:63fce984528cec8714c365919882f8fb64c8a3edf23fdfa0b218a2756125456f
Status: Downloaded newer image for ubuntu:14.04
docker.io/library/ubuntu:14.04
```

### docker build

Create a `Dockerfile` in current folder.

```sh
[vagrant@docker-host ~]$ more Dockerfile
FROM        ubuntu:14.04
MAINTAINER  nareth.im@gmail.com
RUN         apt-get update && apt-get install -y redis-server
EXPOSE      6379
ENTRYPOINT  ["/usr/bin/redis-server"]
```

Use docker build to create a image.

```sh
docker build -t xiaopeng163/redis:0.1 .
```

```sh
[vagrant@docker-host ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
xiaopeng163/redis   0.1                 04ee3d737115        About a minute ago   214MB
ubuntu              14.04               df043b4f0cf1        5 weeks ago          197MB
```

### docker history

```sh
docker history xiaopeng163/redis:0.1
```
Output:

```sh
[vagrant@docker-host ~]$ docker history xiaopeng163/redis:0.1
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
04ee3d737115        3 minutes ago       /bin/sh -c #(nop)  ENTRYPOINT ["/usr/bin/red…   0B                  
443fc1523157        3 minutes ago       /bin/sh -c #(nop)  EXPOSE 6379                  0B                  
e0bdf337fce4        3 minutes ago       /bin/sh -c apt-get update && apt-get install…   17.5MB              
c14899cbbe13        4 minutes ago       /bin/sh -c #(nop)  MAINTAINER nareth.im@gmai…   0B                  
df043b4f0cf1        5 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           5 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B                  
<missing>           5 weeks ago         /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B                  
<missing>           5 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   195kB               
<missing>           10 months ago       /bin/sh -c #(nop) ADD file:276b5d943a4d284f8…   196MB               

```

### docker images

`docker images` will list all avaiable images on your local host.

```sh
[vagrant@docker-host ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
xiaopeng163/redis   0.1                 04ee3d737115        7 minutes ago       214MB
ubuntu              14.04               df043b4f0cf1        5 weeks ago         197MB
```

### docker rmi

Remove docker images.

```sh
```

## 2. Docker Containers

### Start a container in interactive mode

```sh
docker run -i --name test3  ubuntu:14.04
```

Output:

```sh
[vagrant@docker-host ~]$ docker run -i --name test3  ubuntu:14.04
pwd
/
ls -l
total 4
drwxr-xr-x.  2 root root 4096 Dec 17  2019 bin
drwxr-xr-x.  2 root root    6 Apr 10  2014 boot
drwxr-xr-x.  5 root root  340 Oct 28 17:17 dev
drwxr-xr-x.  1 root root   66 Oct 28 17:17 etc
drwxr-xr-x.  2 root root    6 Apr 10  2014 home
drwxr-xr-x. 12 root root  208 Dec 17  2019 lib
drwxr-xr-x.  2 root root   34 Dec 17  2019 lib64
drwxr-xr-x.  2 root root    6 Dec 17  2019 media
drwxr-xr-x.  2 root root    6 Apr 10  2014 mnt
drwxr-xr-x.  2 root root    6 Dec 17  2019 opt
dr-xr-xr-x. 97 root root    0 Oct 28 17:17 proc
drwx------.  2 root root   37 Dec 17  2019 root
drwxr-xr-x.  1 root root   21 Sep 16 22:21 run
drwxr-xr-x.  1 root root   44 Sep 16 22:21 sbin
drwxr-xr-x.  2 root root    6 Dec 17  2019 srv
dr-xr-xr-x. 13 root root    0 Oct 28 16:28 sys
drwxrwxrwt.  2 root root    6 Dec 17  2019 tmp
drwxr-xr-x.  1 root root   18 Dec 17  2019 usr
drwxr-xr-x.  1 root root   17 Dec 17  2019 var
ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:656 (656.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

exit
[vagrant@docker-host ~]$
```

### Start a container in background

Start a container in background using `xiaopeng163/redis:0.1` image, and the name of the container is `demo`. Through `docker ps` we can see all running Containers

```sh
docker run -d --name demo xiaopeng163/redis:0.1

docker ps
```

Output:

```sh
[vagrant@docker-host ~]$ docker run -d --name demo xiaopeng163/redis:0.1
26cfddf939de945e2e954d4d9dcc3a8e6825762ca05b0ea7043e42524b5536de
[vagrant@docker-host ~]$ docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS               NAMES
26cfddf939de        xiaopeng163/redis:0.1   "/usr/bin/redis-serv…"   About a minute ago   Up About a minute   6379/tcp            demo
```

###  stop/remove containers

Sometime, we want to manage multiple containers each time, like start, stop, rm.

Firstly, we can use --filter to filter out the containers we want to manage.

```sh
docker ps -a --filter "status=exited"
```

Output:

```sh
[vagrant@docker-host ~]$ docker ps -a --filter "status=exited"
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
87d5709b23da        ubuntu:14.04        "/bin/bash"         8 minutes ago       Exited (0) 7 minutes ago                       test3
```

Secondly, we can use `-q` option to list only containers ids

```sh
docker ps -aq --filter "status=exited"
```

Output:

```sh
[vagrant@docker-host ~]$ docker ps -aq --filter "status=exited"
87d5709b23da
```

At last, we can batch processing these containers, like remove them all or start them all:

```sh
docker rm $(docker ps -aq --filter "status=exited")
```

Output:

```sh
[vagrant@docker-host ~]$ docker rm $(docker ps -aq --filter "status=exited")
87d5709b23da
```
