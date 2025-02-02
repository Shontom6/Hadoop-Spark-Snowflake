Vagrant.configure("2") do |config|
  # Common settings for all VMs
  config.vm.box = "hashicorp/bionic64"

  # Define VM 1 (node01)
  config.vm.define "vm1" do |vm1|
    vm1.vm.hostname = "node01"
    vm1.vm.network "private_network", ip: "192.168.50.4"

    # Provisioning for node01
    vm1.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y openjdk-11-jdk wget
     
    SHELL
  end

  # Define VM 2 (node02)
  config.vm.define "vm2" do |vm2|
    vm2.vm.hostname = "node02"
    vm2.vm.network "private_network", ip: "192.168.50.5"

    # Provisioning for node02
    vm2.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y openjdk-11-jdk wget
      
    SHELL
  end
end
