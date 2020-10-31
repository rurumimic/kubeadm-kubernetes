# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  config.vm.define vm_name = "lb" do |config|
    config.vm.hostname = "lb"
    config.vm.network :private_network, ip: "192.168.10.101"
  end

  (1..3).each do |i|
    config.vm.define vm_name = "controlplane-#{i}" do |config|
      config.vm.hostname = "controlplane-#{i}"
      config.vm.network :private_network, ip: "192.168.10.#{i+110}"
      config.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
      end
    end
  end

  (1..3).each do |i|
    config.vm.define vm_name = "etcd-#{i}" do |config|
      config.vm.hostname = "etcd-#{i}"
      config.vm.network :private_network, ip: "192.168.10.#{i+120}"
    end
  end

  (1..3).each do |i|
    config.vm.define vm_name = "worker-#{i}" do |config|
      config.vm.hostname = "worker-#{i}"
      config.vm.network :private_network, ip: "192.168.10.#{i+130}"

      config.vm.provision :ansible do |ansible|
        ansible.groups = {
          "controlplane" => ["controlplane-[1:3]"],
          "etcd"  => ["etcd-[1:3]"],
          "worker"  => ["worker-[1:3]"],
        }
        ansible.playbook = "ansible/playbook.yaml"
        ansible.limit = "all"
      end
    end
  end

end