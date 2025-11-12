# Variables
# Image à utiliser
IMAGE_NAME = "bento/ubuntu-20.04"
# RAM
MEM = 2048
# Nombre de CPU
CPU = 2
# Nombre de noeuds
WORKER_NB = 1
# Prefix pour la plage d'adresses IP à utiliser
NODE_NETWORK_BASE = "10.1.1"
# Master IP
MASTER_IP = "#{NODE_NETWORK_BASE}.10"

# TEMP
WORKER_IP = "#{NODE_NETWORK_BASE}.11"

Vagrant.configure("2") do |config|
    # We need to add the IP of both nodes to /etc/hosts
    # TODO: Make this config more dynamic in case of multiple worker nodes.
    config.vm.provision "shell", inline: <<-SHELL
          apt-get update -y
          echo "$MASTER_IP k8s-master" >> /etc/hosts
          echo "$WORKER_IP k8s-worker1" >> /etc/hosts
      SHELL

  config.ssh.insert_key = true

  # Configuration de la RAM et du CPU
  config.vm.provider "virtualbox" do |v|
    v.gui = false
    v.memory = MEM
    v.cpus = CPU
  end

  # Configuration du Master
  config.vm.define "k8s-master" do |master|
    master.vm.box = IMAGE_NAME
    master.vm.network "private_network", ip: MASTER_IP
    master.vm.hostname = "k8s-master"
    # Forward port 8989 in control plane to port 8989 on the host for demo purpose
    master.vm.network "forwarded_port", guest: 8989, host: 8989
  end

  # Configuration du/des Worker(s)
  (1..WORKER_NB).each do |i|
    config.vm.define "k8s-worker#{i}" do |worker|
      worker.vm.box = IMAGE_NAME
      worker.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
      worker.vm.hostname = "k8s-worker#{i}"
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.groups = {
      "master" => ["k8s-master"],
      "worker" => ["k8s-worker[1:#{WORKER_NB}]"],
      "kube_cluster:children" => ["master", "worker"],
      "kube_cluster:vars" => { "master_ip" => MASTER_IP }
    }
  end
end
