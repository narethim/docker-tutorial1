# Single node setup

## Steps

### Start the VM

```sh
cd docker-k8s-lab/lab/docker/single-node/vagrant-centos7

vagrant up
```

Status

```sh
vagrant status

vagrant ssh-config
```

Connect to the VM

```sh
vagrant ssh
```

While inside the VM

```sh
sudo apt update

sudo apt -y upgrade
```

### Teardown the VM

```sh
vagrant halt

vagrant destroy
```
