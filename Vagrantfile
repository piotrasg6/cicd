Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  config.vagrant.plugins = "vagrant-share"
  config.vm.box = "ubuntu/focal64"
  config.vm.synced_folder ".", "/vagrant", :mount_options => ["umask=077"]

  config.vm.define "node" do |node|
    node.vm.hostname = "node"
    node.vm.network "private_network", ip: "10.0.0.20"
    
    node.vm.provider "virtualbox" do |vb|
      vb.name = "node"
      vb.cpus = "1"
      vb.memory = "1024"
    end
  end

  config.vm.define "jenkins" do |jenkins|
    jenkins.vm.hostname = "jenkins"
    jenkins.vm.network "private_network", ip: "10.0.0.10"
    jenkins.vm.network "forwarded_port", guest: 8080, host: 8081
    jenkins.vm.provision "shell", path: "./.vagrant-provision/provision.sh"

    jenkins.vm.provider "virtualbox" do |vb|
      vb.name = "jenkins"
      vb.cpus = "1"
      vb.memory = "2048"
    end
  end
end