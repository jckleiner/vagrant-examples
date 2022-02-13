# Vagrant Multi-Machine Setup For MacOs M1 using Docker as the Provider

> The base Dockerfile and Vagrantfile is taken from https://dev.to/taybenlor/running-vagrant-on-an-m1-apple-silicon-using-docker-3fh4. 
> Check out that article to learn more about running Vagrant with Docker as provider on MacOs M1.

Since Virtualbox is not supported on the M1 Macbooks, using Virtualbox as a Vagrant provider is not possible.
One solution is to use Docker as the provider. There might be other ways to do this (VMWare as the provider?) but this works for me.

This Vagrant script sets up 2 virtual machines `vm1` and `vm2` (Ubuntu 20.04) in a private network using Docker as the provider.
Both machines will have `docker` and `docker-compose` installed 
and both machines will have an `nginx` container running inside them after the provisioning is finished.

### Run / Test 

The docker-compose plugin needs to be installed on the host machine: `vagrant plugin install vagrant-docker-compose`.

After that run `vagrant up`.

After the setup is finished, validate that the Docker containers started:
 * Run `docker ps`
 * 2 containers should be started: 
   * `vm1` with the following ports exposed: `2001->22` and `8061->80`
   * `vm2` with the following ports exposed: `2002->22` and `8062->80`

Test the connection `host -> vm1`: 
 * Send a request to virtual machine 2: `curl -I http://localhost:8061`
 * You should see a 200 OK response

Test the connection `host -> vm2`: 
 * Send a request to virtual machine 2: `curl -I http://localhost:8062`
 * You should see a 200 OK response

Test the connection `vm1 -> vm2`: 
 * Connect to virtual machine 1: `vagrant ssh vm1`
 * Send a request to virtual machine 2: `curl -I http://10.10.10.12`
 * You should see a 200 OK response

Test the connection `vm2 -> vm1`: 
 * Connect to virtual machine 2: `vagrant ssh vm2`
 * Send a request to virtual machine 1: `curl -I http://10.10.10.11`
 * You should see a 200 OK response

### SSH
You can ssh into machines either with 
 * `vagrant ssh <vm-name>` or 
 * `ssh vagrant@127.0.0.1 -p <ssh-port> -o StrictHostKeyChecking=no -i .vagrant/machines/<vm-name>/docker/private_key`

So, to connect to vms using the explicit method:
* Virtual Machine 1: `ssh vagrant@127.0.0.1 -p 2001 -o StrictHostKeyChecking=no -i .vagrant/machines/vm1/docker/private_key`
* Virtual Machine 2: `ssh vagrant@127.0.0.1 -p 2002 -o StrictHostKeyChecking=no -i .vagrant/machines/vm2/docker/private_key`

### Network
This vagrant script will setup a private network and will assign static IP addresses to the virtual machines.
Virtual machine 1 will have the IP `10.10.10.11` and virtual machine 2 `10.10.10.12`. Since it's a private network, the IP's are only usable/resolvable in the virtual machines (or docker containers in our case). Which means you can't access the containers with their IP address from your host machine. You can however access them via the exposed container ports.
