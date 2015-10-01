# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "petergdoyle/CentOS-7-x86_64-Minimal-1503-01"

  #allows an ssh connection to be forwarded through the host machine on 2222
  #config.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "0.0.0.0", id: "ssh", auto_correct: true
  #allows mongo connection to be forwarded through the host machine on 227017

  config.vm.network "forwarded_port", guest: 8000, host: 28000, host_ip: "0.0.0.0", id: "node_port_8000", auto_correct: true

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  #config.vm.synced_folder "mongo/", "/mongo"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
    vb.cpus=4 #recommended=4 if available
    vb.memory = "4096" #recommended=3072 or 4096 if available
  end

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL

  config.vm.provision "shell", inline: <<-SHELL

  yum -y update

  #install additional tools
  yum -y install vim htop curl wget net-tools tree

  #install node.js and npm
  yum -y install epel-release gcc gcc-c++
  yum -y install nodejs npm

  npm install format-json-stream -g

  #install azure-cli
  npm install azure-cli -g

  #setup azure account information
  #https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-command-line-tools/
  sudo -c vagrant 'azure account import /vagrant/azure-account.publishsettings'
  sudo -c vagrant 'azure account list'


  #install docker service
  cat >/etc/yum.repos.d/docker.repo <<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
  yum -y install docker
  systemctl start docker.service
  systemctl enable docker.service

  #allow non-sudo access to run docker commands for user vagrant
  #if you have problems running docker as the vagrant user on the vm (if you 'vagrant ssh'd in
  #after a 'vagrant up'), then
  #restart the host machine and ssh in again to the vm 'vagrant halt; vagrant up; vagrant ssh'
  groupadd docker
  usermod -aG docker vagrant


  #install docker-compose.
  #Compose is a tool for defining and running multi-container applications with Docker.
  yum -y install python-pip
  pip install -U docker-compose

  #eventually this will go in the Dockerfile
  #install kafka
  export KAFKA_HOME='/home/vagrant/kafka/default'
  echo "export KAFKA_HOME=$KAFKA_HOME" >> /etc/profile.d/kafka.sh
  curl -O --insecure http://apache.claz.org/kafka/0.8.2.1/kafka_2.9.1-0.8.2.1.tgz
  tar -xvf kafka_2.9.1-0.8.2.1.tgz
  mkdir -p $KAFKA_HOME
  mv kafka_2.9.1-0.8.2.1 $KAFKA_HOME
  ln -s $KAFKA_HOME/kafka_2.9.1-0.8.2.1 $KAFKA_HOME/default
  chown -R vagrant:vagrant $KAFKA_HOME
  rm -f kafka_2.9.1-0.8.2.1.tgz
  mv $KAFKA_HOME/config/server.properties $KAFKA_HOME/config/server.properties.orig
  cp /vagrant/kafka-single-node-server.properties $KAFKA_HOME/config/server.properties

  #because there will be dockerized kafka instances running on this vm
  #it is necessary for development purposes to expose those container ports to
  #the localhost (host vm) this will expose a kafka_zk_0 and 3 kafka_server_0,1,2
  #to the localhost to interact with the containers. of course these containers
  #would have to be named that way when they are created so look at
  #scripts/docker_run_kafka_server.sh and scripts/docker_run_kafka_zk.sh for
  #details on exact container names required for this to work properly
  cat >>/etc/hosts <<-EOF
127.0.0.1   kafka_server_0 kafka_server_1 kafka_server_2 kafka_zk_0 \
kafka_producer_paras_0 \
kafka_producer_anagrams_0 kafak_consumer_anagrams_0 \
kafka_stats_consumer_paras_0 kafka_stats_consumer_even_0 kafka_stats_consumer_odd_0
::1         kafka_server_0 kafka_server_1kafka_server_2 kafka_zk_0 \
kafka_producer_paras_0 \
kafka_producer_anagrams_0 kafak_consumer_anagrams_0 \
kafka_stats_consumer_paras_0 kafka_stats_consumer_even_0 kafka_stats_consumer_odd_0
EOF



  #set hostname
  hostnamectl set-hostname kafka-streaming.vbx

  SHELL
end
