Vagrant.configure("2") do |config|

  #VM1: Master
  config.vm.define "master" do |master|
    master.vm.box = "centos/7"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.11.10"
    master.ssh.insert_key = false
    master.vm.provider :virtualbox do |v|
      v.name = "master"
      v.customize ["modifyvm", :id, "--memory", 2048]
      v.customize ["modifyvm", :id, "--cpus", 2]
    end
    master.vm.provision :ansible_local do |ansible|
      ansible.playbook = "/vagrant/provisioning/servers.yml"
#      ansible.install = true
#      ansible.version = "latest"
#      ansible.install_mode = "pip3"
      ansible.compatibility_mode = "2.0"
#      ansible.inventory_path = "/vagrant/inventory"
      ansible.config_file = "/vagrant/ansible.cfg"
      ansible.limit = "all"
    end
#    master.vm.provision :shell, :inline => "reboot", run: "always"

    $script = <<-SCRIPT
    echo I start provisioning...
    date > /etc/vagrant_provisioned_at
    yum clean all
    yum makecache	
    yum install -y epel-release python3-pip ansible 
    yum install -y git wget curl vim net-tools bash-completion
    yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum install -y https://yum.puppet.com/puppet6-release-el-7.noarch.rpm
    sed -i '/192.168.11.*/d' /etc/hosts
    echo "192.168.11.10 master.domain.local master" >> /etc/hosts
    echo "192.168.11.11 node1.domain.local node1" >> /etc/hosts
    echo "192.168.11.12 node2.domain.local node1" >> /etc/hosts
    host=$( hostname )
    echo "hostname = [$host] / HOSTNAME = [$HOSTNAME]"
    if [[ $host = master ]]; then
      echo $HOSTNAME
      echo "Puppetmaster" 
      yum install -y puppetserver puppet-agent
      sed -i 's/\-Xms2g \-Xmx2g/\-Xms512m \-Xmx512m/' /etc/sysconfig/puppetserver
      #puppet resource service puppetserver ensure=running enable=true
    else
      echo "Puppet Agent"
      yum install -y puppet-agent
      #puppet resource service puppet ensure=running enable=true
    fi
    SCRIPT

    #Vagrant.configure("2") do |config|
      # if vagrant-vbguest is installed stop auto updating virtualbox guest add-on update
      if defined? VagrantVbguest
        config.vbguest.auto_update = false
      end
      config.vm.provision "shell", inline: $script
    #end
    # Ansible provisioner.
    # config.vm.provision "ansible" do |ansible|
    #   ansible.compatibility_mode = "2.0"
    #   ansible.playbook = "provisioning/playbook.yml"
    #   ansible.inventory_path = "provisioning/inventory"
    #    ansible.become = true
    #end
  end

  # configure two machines server1 & server2
  (1..2).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = "node#{i}"
      node.vm.network :private_network, ip: "192.168.11.1#{i}"
      node.ssh.insert_key = false
      node.vm.provider :virtualbox do |v|
        v.name = "node#{i}"
        v.customize ["modifyvm", :id, "--memory", 2048]
        v.customize ["modifyvm", :id, "--cpus", 2]
      end
    end
  end
end

