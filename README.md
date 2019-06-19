# Vagrant testing swarm cluster

Fully configurable manager/workers docker swarm VMs setup, included is a portainer stack yml if needed deployment

Arch linux ;) based

## How to use
1. Install and configure Vagrant [Vagrant docs](https://www.vagrantup.com/docs/index.html)
2. `cd` into this directory `cd ./vagrant-docker-swarm`
3. Edit `Vagrantfile` to tweak config (ex: cpu, memory, node count etc)
4. run vagrant agent to standup the cluster `vagrant up` (might take some time ...)
5. check cluster status once all is complete `vagrant status`
6. SSH into any node by issuing command `vagrant ssh [node name]`, if node name is omitted primary node will be used (manager node is the primary node)
7. (OPTIONAL) deploy [portainer](https://www.portainer.io/) to manage the cluster with a nice UI exposed at port `9000`

## FAQ
> How do I access the docker container running inside the VMs

This is done by making sure you have a port binding from container to host `-p host:container` and accessing the container on the defined network from the `Vagrantfile` (`ip_mask`)

> How to deploy Portainer

Portainer will be deployed as a stack using the included docker compose yml with steps below
  - SSH into the manager node `vagrant ssh manager-1`
  - navigate to host shared folder `cd /vagrant`
  - deploy stack from file `docker stack deploy --compose-file=portainer-agent-stack.yml portainer`
