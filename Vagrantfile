# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'json'

vconfig = JSON.parse(File.read(File.join(File.dirname(__FILE__), 'json/vconfig.json')))

Vagrant.configure("2") do |config|

  vconfig.each do |vcfg|
    config.vm.define vcfg['name'] do |vc|
      vc.vm.box = vcfg['box']
      vc.vm.hostname = vcfg['name']
      vc.vm.network "forwarded_port", guest: vcfg['guest_port'], host: vcfg['host_port']
      vc.vm.network "private_network", ip: vcfg['ip_addr']
      # for ssh from ext hosts
      vc.vm.network :forwarded_port, guest: 22, host: 2222, host_ip: "localhost", id: "ssh", auto_correct: true
      vc.vm.provider vcfg['vm'] do |vb|
        vb.memory = vcfg['memory']
        vb.cpus = vcfg['cpus']
      end

      # install java for jenkins worker node
      config.vm.provision "shell", inline: "java -version || { apt-get update; apt-get install openjdk-8-jre-headless -y;}"

      config.vm.provision "shell", inline: "python --version || { apt-get update; apt-get install python -y;}"

      # install docker using ansible role
      vc.vm.provision "ansible_local" do |ansible|
        ansible.limit = "all"
        ansible.playbook = vcfg['playbook']
        ansible.inventory_path = vcfg['hosts']
        ansible.install_mode = "pip"
      end

      # add jenkins user
      vc.vm.provision "shell", inline: "sudo groupadd --system jenkins; sudo useradd -m --system -g jenkins jenkins; sudo usermod -aG docker jenkins; sudo usermod -aG sudo jenkins;"

      # add swap
      vc.vm.provision :shell, inline: "fallocate -l 4G /swapfile; chmod 0600 /swapfile; mkswap /swapfile; swapon /swapfile; echo '/swapfile none swap sw 0 0' >> /etc/fstab"

      # adjust cache pressure
      vc.vm.provision :shell, inline: "echo vm.swappiness = 10 >> /etc/sysctl.conf; echo vm.vfs_cache_pressure = 50 >> /etc/sysctl.conf; sysctl -p"


      # enable docker remote API
      vc.vm.provision "shell", inline: "sudo sed -i -e '/^ExecStart=/s/$/ -H tcp:\\/\\/0.0.0.0:4243/' /lib/systemd/system/docker.service"
      # add jenkins docker slave image
      vc.vm.provision "shell", inline: "docker pull jenkins/jnlp-slave"
    end
  end
end
