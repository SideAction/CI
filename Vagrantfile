# -*- mode: ruby -*-
# vi: set ft=ruby :
network = "192.168.77"
server_ip = "#{network}.66"
nexus_ip = "#{network}.67"
box = "bento/ubuntu-16.04"
N = 2

Vagrant.configure("2") do |config|

  config.vm.define "nexus" do |nexus|
    nexus.vm.box = box
    nexus.ssh.forward_agent = true
    nexus.vm.boot_timeout = 120

    nexus.vm.hostname = "nexus"
    nexus.vm.network "private_network", ip: "#{nexus_ip}"
    nexus.vm.network "forwarded_port", guest: 443, host: 8155, host_ip: "127.0.0.1"

    nexus.vm.synced_folder "./", "/vagrant"
    nexus.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
    end

    nexus.vm.provision "shell", inline: "apt-get update; apt-get upgrade -y"

    nexus.vm.provision "ansible" do |ansible|
      ansible.extra_vars = {}
      ansible.become = true
      ansible.limit = "all"
      ansible.verbose = "v"
      ansible.extra_vars = {remote_user: "vagrant"}
      ansible.playbook = "nexus.yml"
    end
  end

  config.vm.define "gocdserver" do |server|
    server.vm.box = "bento/ubuntu-16.04"
    server.ssh.forward_agent = true
    server.vm.boot_timeout = 120

    server.vm.hostname = "gocd.server"
    server.vm.network "private_network", ip: "#{server_ip}"
    server.vm.network "forwarded_port", guest: 8154, host: 8154, host_ip: "127.0.0.1"

    server.vm.synced_folder "./", "/vagrant"
    server.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
    end

    server.vm.provision "ansible" do |ansible|
      ansible.groups = {
          "server" => ["default"],
          "agents" => ["default"]
      }
      ansible.extra_vars = {
          GOCD_ADMIN_EMAIL: 'jcarlson@gmail.com'
      }
      ansible.limit = "all"
      ansible.verbose = "v"
      ansible.extra_vars = {remote_user: "vagrant"}
      ansible.playbook = "server.yml"
    end
  end

  (1..N).each do |agent_id|
    config.vm.define "agent#{agent_id}" do |agent|
      agent.vm.hostname = "agent#{agent_id}"
      agent.vm.box = "bento/ubuntu-16.04"
      agent.ssh.forward_agent = true
      agent.vm.boot_timeout = 120

      agent.vm.hostname = "gocd.agent#{agent_id}"
      agent.vm.network "private_network", ip: "#{network}.#{20+agent_id}"

      agent.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
      end

      # Only execute once the Ansible provisioner,
      # when all the agents are up and ready.
      if agent_id == N
        agent.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the agents
          ansible.limit = "all"
          ansible.playbook = "agents.yml"
          ansible.extra_vars = {"remote_user": "vagrant"}

          ansible.groups = {
              "server" => ["default"],
              "agents" => ["default"]
          }
          ansible.extra_vars = {
              GOCD_ADMIN_EMAIL: 'jcarlson@gmail.com',
              GOCD_SERVER_HOST: server_ip
          }
        end
      end
    end
  end

end

