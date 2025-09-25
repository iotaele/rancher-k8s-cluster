# -*- mode: ruby -*-  
# vi: set ft=ruby

Vagrant.configure("2") do |config|
  config.vm.define "k3s-master" do |master|
    master.vm.box = "generic/ubuntu2204" # Or your preferred base box
    master.vm.provider "qemu" do |qemu|
      qemu.memory = 4096
      qemu.cpus = 2
    end
    master.vm.network "private_network", ip: "192.168.56.10"
    master.dns.tld = "devops"
    master.dns.patterns = [/^(\w+\.)curso.devops$/, /^curso.devops&/]
    master.vm.network "forwarded_port", guest: 80, host: 80
    master.vm.network "forwarded_port", guest: 443, host: 443
    master.vm.hostname = "k3s-master"
    master.vm.provision "shell", inline: <<-SHELL
      curl -sfL https://get.k3s.io | sh -s - server --write-kubeconfig-mode 644
      mkdir /home/vagrant/.kube
      touch /home/vagrant/.kube/config
      sudo cp /etc/rancher/k3s/k3s.yaml /home/vagrant/.kube/config
      sudo chown vagrant:vagrant /home/vagrant/.kube/config
      sudo apt update
      sudo apt install podman -y
    SHELL
  end

  # Add agent nodes 
  (1..3).each do |i|
    config.vm.define "k3s-agent-#{i}" do |agent|
      agent.vm.box = "generic/ubuntu2204"
      agent.vm.provider "qemu" do |qemu|
        qemu.memory = 2048
        qemu.cpus = 2
      end
      agent.vm.network "private_network", ip: "192.168.56.1#{i}"
      agent.dns.tld = "devops"
      agent.dns.patterns = [/^(\w+\.)curso.devops$/, /^curso.devops&/]
      agent.vm.hostname = "k3s-agent-#{i}"
      agent.vm.provision "shell", inline: <<-SHELL
        sudo apt update
        sudo apt install podman -y
        K3S_URL=https://192.168.56.10:6443 K3S_TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token) curl -sfL https://get.k3s.io | sh -
      SHELL
    end
  end
end
