# -*- mode: ruby -*-
# vi: set ft=ruby :

$config = ''

Vagrant.configure("2") do |config|
  $config = config
  config.vm.box_check_update = false

  #config.ssh.private_key_path = '~/.ssh/id_rsa'
  #config.ssh.forward_agent = true

  # Sync time with the local host
  config.vm.provider 'virtualbox' do |vb|
    vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
  end

  def createMaster
    $master_instances.each{|key, value|
      exec key, value
    }
  end

  def createNode
    $node_instances.each{|key, value|
      exec key, value
    }
  end

  def exec(name, ip)
    $config.vm.define name do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = name
    node.vm.network "private_network", ip: ip
    node.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)", auto_config: true
      
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 1
      vb.name = name
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
          yum install -y wget curl conntrack-tools vim net-tools socat ntp kmod ceph-common lrzsz bind-utils
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
          echo 'install docker'
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
        
          echo '安装docker-compose'
          yum -y install epel-release
          yum -y install python-pip
          pip install --upgrade pip
          pip install docker-compose --ignore-installed requests

        SHELL
    end
  end
end

# 创建两台master机器
$master_instances = {
	"master01"=>"172.17.8.101",
	"master02"=>"172.17.8.102"
}

# 创建两台node机器
$node_instances = {
	"node01"=>"172.17.8.103",
	"node02"=>"172.17.8.104"
}

# ruby多线程不好用，不支持并行执行，慢慢装吧
createMaster
createNode

end






