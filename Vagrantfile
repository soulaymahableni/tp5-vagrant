# vi: ft=ruby:ts=2:ai:sw=2
LAB_VMS = [
  { name: "lab1", ip: "192.168.56.11", prefix: 1 },
  { name: "lab2", ip: "192.168.56.12", prefix: 2 },
]

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"

  LAB_VMS.each do |vm|
    config.vm.define vm[:name] do |node|
      node.vm.hostname = vm[:name]
      node.vm.network "private_network", ip: vm[:ip]

      # Accès depuis le PC : 1xxxx pour lab1, 2xxxx pour lab2
      node.vm.network "forwarded_port", guest: 8080, host: "#{vm[:prefix]}8080".to_i, host_ip: "0.0.0.0"  # Jenkins
      node.vm.network "forwarded_port", guest: 9443, host: "#{vm[:prefix]}9443".to_i, host_ip: "0.0.0.0"  # Portainer
      node.vm.network "forwarded_port", guest: 8082, host: "#{vm[:prefix]}8082".to_i, host_ip: "0.0.0.0"  # site web

      node.vm.provider "virtualbox" do |vb|
        vb.name   = "tp5-#{vm[:name]}"
        vb.memory = 2048
        vb.cpus   = 2
      end

      # Provisionnement avec Ansible et les rôles du TP4
      node.vm.provision "ansible" do |a|
        a.playbook = "playbook.yml"
        a.compatibility_mode = "2.0"
      end
    end
  end
end
