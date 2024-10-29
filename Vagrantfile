Vagrant.configure("2") do |config|

config.vm.define "ansible" do |ansible|
  ansible.vm.box = "generic/ubuntu2204"
  ansible.vm.network "public_network"
  ansible.vm.network "private_network", ip: "10.11.12.21", virtualbox__intnet: "ansible_lab"
  ansible.vm.hostname = "ansible"
  ansible.vm.synced_folder "./ansible_data", "/vagrant_data"
  ansible.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]
	vb.name = "ANSIBLE"
    end
  ansible.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git ansible
  SHELL
  end
config.vm.define "ubuntu" do |ubuntu|
  ubuntu.vm.box = "generic/ubuntu2204"
  ubuntu.vm.network "public_network"
  ubuntu.vm.network "private_network", ip: "10.11.12.22", virtualbox__intnet: "ansible_lab"
  ubuntu.vm.hostname = "ubuntu"
  ubuntu.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]
	vb.name = "UBUNTU"
    end
  end
config.vm.define "debian" do |debian|
  debian.vm.box = "generic/debian11"
  debian.vm.network "public_network"
  debian.vm.network "private_network", ip: "10.11.12.23", virtualbox__intnet: "ansible_lab"
  debian.vm.hostname = "debian"
  debian.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]
	vb.name = "DEBIAN"
    end
  end
config.vm.define "oracle" do |oracle|
  oracle.vm.box = "generic/oracle9"
  oracle.vm.network "public_network"
  oracle.vm.network "private_network", ip: "10.11.12.24", virtualbox__intnet: "ansible_lab"
  oracle.vm.hostname = "oracle"
  oracle.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]
	vb.name = "ORACLE"
    end
  end
end