# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Sync time with the local host
  config.vm.provider 'virtualbox' do |vb|
   vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
  end

  # 创建两台master、两台node机器
  $num_instances = 4
  # 机器名称
  $name = ''

  # curl https://discovery.etcd.io/new?size=3

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

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

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  (1..$num_instances).each do |i|

    #config.ssh.private_key_path = '~/.ssh/id_rsa'
    #config.ssh.forward_agent = true

    # 设置两台master和node机器名称
    if i == 1 or i == 2
      $name = "master%02d" % i
    else
      $name = "node%02d" % i
    end

    config.vm.define $name do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = $name
    ip = "172.17.8.#{i+100}"
    node.vm.network "private_network", ip: ip
    node.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)", auto_config: true
    #node.vm.synced_folder "/Users/DuffQiu/share", "/home/vagrant/share"

    node.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
      vb.memory = "3072"
      vb.cpus = 1
      vb.name = $name
    end

    node.vm.provision "shell" do |s|
      s.inline = <<-SHELL
        # change time zone
        cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
        timedatectl set-timezone Asia/Shanghai
        rm /etc/yum.repos.d/CentOS-Base.repo
        cp /vagrant/yum/*.* /etc/yum.repos.d/
        mv /etc/yum.repos.d/CentOS7-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo
        # using socat to port forward in helm tiller
        # install  kmod and ceph-common for rook
        yum install -y wget curl conntrack-tools vim net-tools socat ntp kmod ceph-common
        # enable ntp to sync time
        echo 'sync time'
        systemctl start ntpd
        systemctl enable ntpd
        echo 'disable selinux'
        setenforce 0
        sed -i 's/=enforcing/=disabled/g' /etc/selinux/config
echo 'enable iptable kernel parameter'
cat >> /etc/sysctl.conf <<EOF
net.ipv4.ip_forward=1
EOF
sysctl -p
echo 'set host name resolution'
cat >> /etc/hosts <<EOF
172.17.8.101 master01
172.17.8.102 master02
172.17.8.103 node01
172.17.8.104 node02
EOF
        cat /etc/hosts
        echo 'set nameserver'
	echo "nameserver 8.8.8.8">/etc/resolv.conf
	cat /etc/resolv.conf
        echo 'disable swap'
        swapoff -a
        sed -i '/swap/s/^/#/' /etc/fstab
        #create group if not exists
        egrep "^docker" /etc/group >& /dev/null
        if [ $? -ne 0 ]
        then
          groupadd docker
        fi
        usermod -aG docker vagrant
        rm -rf ~/.docker/
        echo '安装docker'
        yum install -y yum-utils device-mapper-persistent-data lvm2
        yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        yum install -y docker-ce
        mkdir -p /etc/docker

cat << EOF > /etc/docker/daemon.json
{
"registry-mirrors": [ "http://adf24e46.m.daocloud.io"]
}
EOF

        echo 'enable docker'
        systemctl enable docker
        systemctl start docker
        docker info
      
       # echo '安装docker-compose'
       # curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
       # chmod +x /usr/local/bin/docker-compose

      SHELL
      end
    end
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end