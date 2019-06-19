# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Author:  Najeeb Rifaat (mohamme89d@gmail.com)
# Purpose: Vagrant config to spin up full infrastructure of docker swarm
#          manager and any number of workers
# Usage:   Update flags below to configure little details
#          then standup the cluster
#
#          ```vagrant up```

# Number of total nodes manager + worker (ex: 3 => 1 manager + 2 workers)
nodes = 3

# Amount of memory assigned to each VM (unit is MB)
memory = 512

# Number of CPUs assigned to each VM (cores NOT threads!)
cpus = 1

# IP mask used for all nodes switch
# This IP will be exposed to the host ('XX' will be replaced)
ip_mask = "192.168.10.XX"

# Build up manager config
manager = { :name => "manager-1", :ip => ip_mask.sub("XX", "10") }

# VM image default user
user = "vagrant"

# Build up workers config
workers = []
(1 .. (nodes - 1)).each do |n|
  workers.push({ :name => "worker-#{n}", :ip => ip_mask.sub("XX", "#{n+10}") })
end

# Start configuring the cluster
Vagrant.configure("2") do |config|

  # Set a baseline VM config for all nodes
  config.vm.provider "virtualbox" do |v|
    v.memory = memory
    v.cpus = cpus
  end

  # Configure manager & save docker swarm join token
  config.vm.define manager[:name], primary: true do |man|
    man.vm.box = "archlinux/archlinux"
    man.vm.hostname = manager[:name]
    man.vm.network "forwarded_port", guest: 80, host: 8080
    man.vm.network "private_network", ip: "#{manager[:ip]}"
    man.vm.provision "shell", inline: <<-SHELL
      echo "### Manager setup: Start ###"
      pacman-key --refresh-keys
      pacman -Sy -q --noconfirm docker
      usermod -aG docker #{user}
      systemctl enable docker.service
      systemctl start docker.service
      docker swarm init --advertise-addr #{manager[:ip]}
      docker swarm join-token -q worker > /vagrant/worker_token
      echo "### Manager setup: Finish ###"
    SHELL
  end

  # Loop to configure on remaining nodes (workers - 1 [manager])
  # And join swarm as workers
  workers.each do |worker|
    config.vm.define worker[:name] do |node|
      node.vm.box = "archlinux/archlinux"
      node.vm.hostname = worker[:name]
      node.vm.network "private_network", ip: "#{worker[:ip]}"
      node.vm.provision "shell", inline: <<-SHELL
        echo "### Worker (#{worker[:name]}) setup: Start ###"
        pacman-key --refresh-keys
        pacman -Sy -q --noconfirm docker
        usermod -aG docker #{user}
        systemctl enable docker.service
        systemctl start docker.service
        docker swarm join \
        --advertise-addr #{worker[:ip]} \
        --listen-addr #{worker[:ip]}:2377 \
        --token $(cat /vagrant/worker_token) #{manager[:ip]}:2377
        echo "### Worker (#{worker[:name]}) setup: Finish ###"
      SHELL
    end
  end

end
