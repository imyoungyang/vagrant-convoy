# -*- mode: ruby -*-
# vi: set ft=ruby :
servers=[
  {
    :hostname => "node0",
    :ip => "192.168.10.20"
  }
 #  ,{
 #   :hostname => "elastic1",
 #   :ip => "192.168.100.21"
 # },{
 #   :hostname => "elastic2",
 #   :ip => "192.168.100.22"  
 # }
]

$script = <<SCRIPT
  sudo su root
  yum check-update
  yum makecache fast

  ## add docker repo to get correct docker-ce version
  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

  ## install related packages.
  yum install -y net-tools git wget curl python-pip telnet vim device-mapper-persistent-data lvm2 docker-ce
  
  ## install docker-compose
  curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  chmod +x /usr/bin/docker-compose
  useradd -g docker docker

  sysctl -w vm.max_map_count=262144
  ulimit -n 65535
  ulimit -l unlimited

  ## increase docker ulimit
  ## ship default at /usr/lib/systemd/system/docker.service
  ## Drop-In put at /etc/systemd/system/
  mkdir /etc/systemd/system/docker.service.d
  echo "[Service]" >> /etc/systemd/system/docker.service.d/increase-ulimit.conf
  echo "LimitMEMLOCK=infinity" >> /etc/systemd/system/docker.service.d/increase-ulimit.conf

  ## Vagrant data for elasticsearch
  groupadd -f -g 1000 elasticsearch && useradd elasticsearch -ou 1000 -g 1000
  
  ## install convoy
  wget https://github.com/rancher/convoy/releases/download/v0.5.0/convoy.tar.gz
  tar xvzf convoy.tar.gz
  cp convoy/convoy convoy/convoy-pdata_tools /usr/bin/
  rm -rf convoy convoy.tar.gz
  mkdir -p /etc/docker/plugins/
  bash -c 'echo "unix:///var/run/convoy/convoy.sock" > /etc/docker/plugins/convoy.spec'

  ## setup loopback device. change it later in production mode
  truncate -s 1G data.vol
  truncate -s 1G metadata.vol
  losetup /dev/loop5 data.vol
  losetup /dev/loop6 metadata.vol

  ## setup Convoy plugins daemon
  convoy daemon --drivers devicemapper --driver-opts dm.defaultvolumesize=1G --driver-opts dm.datadev=/dev/loop5 --driver-opts dm.metadatadev=/dev/loop6

  ## start docker daemon
  systemctl daemon-reload
  systemctl restart docker

  ## Configure Docker to start on boot
  systemctl enable docker

  # ## join docker swarm
  # if [ "$(hostname)" == "elastic0" ];then
  #   docker swarm init --advertise-addr=192.168.100.20
  #   docker swarm join-token manager -q > /vagrant/token
  # fi
  # if [ "$(hostname)" == "elastic1" ];then
  #   docker swarm join --token $(cat /vagrant/token) --advertise-addr=192.168.100.21 192.168.100.20:2377
  # fi
  # if [ "$(hostname)" == "elastic2" ];then
  #   docker swarm join --token $(cat /vagrant/token) --advertise-addr=192.168.100.22 192.168.100.20:2377
  # fi
SCRIPT

# This defines the version 2 of vagrant
Vagrant.configure(2) do |config|
  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = machine[:hostname]
      node.vm.network "private_network", ip: machine[:ip]
      config.vm.synced_folder ".", "/vagrant", type: "nfs", :linux__nfs_options => ["no_root_squash"], :map_uid => 0, :map_gid => 0
      node.vm.provider "virtualbox" do |vb|
        vb.memory = 2048
        vb.cpus = 1
      end
      node.vm.provision "shell" do |s|
        s.inline = $script
      end
    end
  end
end
