# Build a Base Image from Scratch

we will build a hello world base image from Scratch.

## System Environment

Docker running on centos 7 and the version

```sh
[vagrant@docker-host ~]$ docker version
Client: Docker Engine - Community
 Version:           19.03.13
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        4484c46d9d
 Built:             Wed Sep 16 17:03:45 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.13
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       4484c46d9d
  Built:            Wed Sep 16 17:02:21 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.3.7
  GitCommit:        8fba4e9a7d01810a393d5d25a3621dc101981175
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

install requirements:

```sh
sudo yum install -y gcc glibc-static
```

## Create a Hello world

Create a `hello.c` and save

```sh
[vagrant@docker-host hello-world]$ pwd
/home/vagrant/hello-world
[vagrant@docker-host hello-world]$ more hello.c
#include <stdio.h>

int main()
{
    printf("hello docker\n");
}

[vagrant@docker-host hello-world]$ 

```

Compile the `hello.c` source file to an binary file, and run it.

```sh
[vagrant@docker-host hello-world]$ gcc -o hello -static  hello.c
[vagrant@docker-host hello-world]$ ls
Dockerfile  hello  hello.c
[vagrant@docker-host hello-world]$ ./hello
hello docker
```

## Build Docker image

Create a Dockerfile like this:

```sh
$ more Dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]
```

build image through:

```sh
[vagrant@docker-host hello-world]$ docker build -t narethim/hello-world .
Sending build context to Docker daemon  864.8kB
Step 1/3 : FROM scratch
 ---> 
Step 2/3 : ADD hello /
 ---> 0a5acadb9992
Step 3/3 : CMD ["/hello"]
 ---> Running in 64f0dff47a9e
Removing intermediate container 64f0dff47a9e
 ---> 87e3d6ab84d8
Successfully built 87e3d6ab84d8
Successfully tagged narethim/hello-world:latest
```

```sh
[vagrant@docker-host hello-world]$ docker image ls
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
narethim/hello-world   latest              87e3d6ab84d8        44 seconds ago      861kB
xiaopeng163/redis      0.1                 04ee3d737115        48 minutes ago      214MB
ubuntu                 14.04               df043b4f0cf1        5 weeks ago         197MB
```

## Run the hello world container

```sh
[vagrant@docker-host hello-world]$ docker run narethim/hello-world
hello docker
```

Done!
