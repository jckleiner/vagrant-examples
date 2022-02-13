# Vagrant Multi-Machine Setup For MacOs M1

> The base Dockerfile and Vagrantfile taken from https://dev.to/taybenlor/running-vagrant-on-an-m1-apple-silicon-using-docker-3fh4.


### SSH
You can ssh into machines either with 
 * `vagrant ssh <vm-name>` or 
 * `ssh vagrant@127.0.0.1 -p <ssh-port> -o StrictHostKeyChecking=no -i .vagrant/machines/<vm-name>/docker/private_key`

So, to manually connect to nodes:
* Node 1: `ssh vagrant@127.0.0.1 -p 2001 -o StrictHostKeyChecking=no -i .vagrant/machines/node1/docker/private_key`
* Node 2: `ssh vagrant@127.0.0.1 -p 2002 -o StrictHostKeyChecking=no -i .vagrant/machines/node2/docker/private_key`

### Network

This vagrant script will setup a private network and will assign static IP addresses to nodes.
Node 1 will have the IP `172.20.128.11` and node 2 `172.20.128.12`. Since it's a private network, the IP's are only usable/resolvable in the virtual machines (or docker containers in our case). Which means you can't access the containers with their IP address from your host machine. You can however access them via the exposed container ports.

Test the connection `host -> node1`: 
 * Send a request to node 2: `curl -I http://localhost:8061`
 * You should see a 200 OK response

Test the connection `host -> node2`: 
 * Send a request to node 2: `curl -I http://localhost:8062`
 * You should see a 200 OK response

Test the connection `node1 -> node2`: 
 * Connect to node 1: `vagrant ssh node1`
 * Send a request to node 2: `curl -I http://172.20.128.12`
 * You should see a 200 OK response

Test the connection `node2 -> node1`: 
 * Connect to node 2: `vagrant ssh node2`
 * Send a request to node 1: `curl -I http://172.20.128.11`
 * You should see a 200 OK response