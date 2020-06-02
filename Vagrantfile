# -*- mode: ruby -*-
# vi: set ft=ruby :

# Configurable parameters
IMAGE = "ubuntu/bionic64"
PKG_DEPS = "ansible docker.io openjdk-11-jre-headless"


Vagrant.configure("2") do |config|
  # Plugins
  # Plugins should be automatically installed if missing.
  config.vagrant.plugins = [
    "vagrant-hosts",
    "vagrant-auto_network",
    "vagrant-vbguest",
  ]
  config.trigger.before :up do |trigger|
    trigger.name = "Init step"
    trigger.info = "I am running before vagrant up!!"
  end
  config.vm.define :minikube do |test_host|
    test_host.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
    end
    test_host.vm.hostname = "minikube.test.local"
    test_host.vm.box = IMAGE
    test_host.vm.network :private_network, :auto_network => true

    test_host.vm.synced_folder "../", "/home/vagrant/service/"
    test_host.vm.provision :shell, inline: "apt-get update && apt-get -y install " + PKG_DEPS + " 1>/dev/null 2>&1"
    test_host.vm.provision :shell, inline: "usermod -aG docker vagrant"
    config.vm.network "forwarded_port", guest: 30000, host: 30000
    config.vm.network "forwarded_port", guest: 30001, host: 30001

    test_host.trigger.after :up do |trigger|
      trigger.name = "Creating the full environment"
      trigger.info = "I am running after vagrant up!!"
      trigger.run_remote = { inline: "su - vagrant -c 'cd /home/vagrant/service/ansible_code/minikube;
       ansible-playbook setup.yml'" }
    end
  end
end
