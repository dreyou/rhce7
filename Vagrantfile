# -*- mode: ruby -*-
# vi: set ft=ruby :
#
Vagrant.configure(2) do |config|
  # 
  # Vagrant boxes for libvirt or virtualbox
  # 
  #config.vm.box = "puppetlabs/centos-7.0-64-nocm"
  config.vm.box = "bento/centos-7.1"
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end
  config.ssh.forward_x11  = true
#
# 
#
  config.vm.define :server1 do |server1|
    server1.ssh.forward_x11  = true
    server1.vm.network "private_network", ip: "192.168.123.210"
#    server1.vm.network "private_network", type: "dhcp"
#    server1.vm.network "private_network", type: "dhcp"
    server1.vm.hostname = "server1.example.com"
    server1.vm.synced_folder "./common", "/vagrant", type: "rsync"
    server1.vm.provision "shell", inline: $common
    server1.vm.provision "shell", inline: $server1
  end
  config.vm.define :server2 do |server2|
    server2.ssh.forward_x11  = true
    server2.vm.network "private_network", ip: "192.168.123.220"
#    server2.vm.network "private_network", type: "dhcp"
#    server2.vm.network "private_network", type: "dhcp"
    server2.vm.hostname = "server2.example.com"
    server2.vm.synced_folder "./common", "/vagrant", type: "rsync"
    server2.vm.provision "shell", inline: $common
    server2.vm.provision "shell", inline: $server2
  end
  config.vm.define :ipa do |ipa|
    ipa.vm.network "private_network", ip: "192.168.123.200"
    ipa.vm.hostname = "ipa.example.com"
    ipa.vm.synced_folder "./common", "/vagrant", type: "rsync"
    ipa.vm.provision "shell", inline: $common
    ipa.vm.provision "shell", inline: $ipa
  end
#
# Common node provisioning
#
$common = <<SCRIPT
#!/bin/sh
>&2 echo Common setup
#
# Check internet connection
#
ping -c 2 -W 2 google-public-dns-a.google.com
if [[ $? != 0 ]]
then
  echo "Can't connect to internet" >&2
  exit 1
fi
#
# Turning off firewalld (clean iptables rules)
#
systemctl stop firewalld.service
systemctl disable firewalld.service
#
# Turn on our interfaces, wich may not be started by vagrant
#
systemctl restart NetworkManager.service
systemctl stop network.service
systemctl start network.service
chkconfig network on
yum -y --disableplugin=fastestmirror install epel-release xorg-x11-xauth mc vim expect
SCRIPT
#
# ipa node provisioning
#
$ipa = <<SCRIPT
#!/bin/sh
>&2 echo Ipa setup
#
sed -i 's/ipa.example.com//' /etc/hosts
sed -i 's/ipa//' /etc/hosts
echo -e "192.168.123.200 ipa.example.com\n" >> /etc/hosts
echo -e "192.168.123.220 server2.example.com server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.example.com server1\n" >> /etc/hosts
systemctl restart NetworkManager

yum -y --disableplugin=fastestmirror update

yum -y --disableplugin=fastestmirror install ipa-server bind-dyndb-ldap

systemctl isolate multi-user.target

expect -f /vagrant/ipa-server.exp 

yum -y --disableplugin=fastestmirror install vsftpd
systemctl enable vsftpd
systemctl start vsftpd
cp -v ~/cacert.p12 /var/ftp/pub

echo "password" | kinit admin
klist

echo "password" | ipa user-add lisa --first=lisa --last=jones --password
echo "password" | ipa user-add linda --first=linda --last=thomsen --password

ipa host-add --force server1.example.com
ipa host-add --force server2.example.com

ipa service-add --force nfs/server1.example.com
ipa service-add --force nfs/server2.example.com
ipa service-add --force cifs/server1.example.com
ipa service-add --force cifs/server2.example.com

SCRIPT
#
# server1 node provisioning
#
$server1 = <<SCRIPT
#!/bin/sh
>&2 echo Server1 setup
sed -i 's/server1.example.com//' /etc/hosts
sed -i 's/server1//' /etc/hosts
echo -e "192.168.123.220 server2.example.com server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.example.com server1\n" >> /etc/hosts
echo -e "192.168.123.200 ipa.example.com ipa\n" >> /etc/hosts
systemctl restart NetworkManager
yum -y groupinstall "Server with GUI"
yum -y update
systemctl enable firewalld.service
systemctl start firewalld.service
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
#yum -y --disableplugin=fastestmirror install ipa-client
#ipa-getkeytab -s ipa.example.com -p nfs/server1.example.com -k /etc/krb5.keytab
SCRIPT
#
# server2 node provisioning
#
$server2 = <<SCRIPT
#!/bin/sh
>&2 echo Server2 setup
sed -i 's/server2.example.com//' /etc/hosts
sed -i 's/server2//' /etc/hosts
echo -e "192.168.123.220 server2.example.com server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.example.com server1\n" >> /etc/hosts
echo -e "192.168.123.200 ipa.example.com ipa\n" >> /etc/hosts
systemctl restart NetworkManager
yum -y groupinstall "Server with GUI"
yum -y update
systemctl enable firewalld.service
systemctl start firewalld.service
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
#yum -y --disableplugin=fastestmirror install ipa-client
#ipa-getkeytab -s ipa.example.com -p nfs/server2.example.com -k /etc/krb5.keytab
SCRIPT
end