# Docker Network Overview

Image reference from [1](https://blog.docker.com/2015/04/docker-networking-takes-a-step-in-the-right-direction-2/)

When you install Docker, it creates three networks automatically. You can list these networks using the docker network ls command:

```sh
[vagrant@docker-host hello-world]$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
81895eac88f6        bridge              bridge              local
115594872e39        host                host                local
b9ec37c530f2        none                null                local
```

## Reference

[Networking overview](https://blog.docker.com/2015/04/docker-networking-takes-a-step-in-the-right-direction-2/)
