# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Use Centos 7 VMs
  config.vm.box = "centos/7"

  ## Expose the VM on the public network
  #config.vm.network "public_network"
  
  # Configure a private network
  config.vm.network "private_network", ip: "192.168.123.10"

  # Configuration for Virtualbox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
  end

  # Configuration for KVM
  config.vm.provider "libvirt" do |vb|
    vb.memory = "512"
  end

  # Generate a random string to be used in the hostname
  variable_string = (0...5).map { ('a'..'z').to_a[rand(26)] }.join

  # VM definition
  config.vm.define "webserver" do |web|
    web.vm.hostname = "iptables-#{variable_string}"
  end
  
  # Provision with shell script
  config.vm.provision "shell", inline: <<-SHELL
    echo "== Updating the OS =="
    yum -y distro-sync

    echo "== Install packages =="
    yum install -y bind-utils httpd mod_ssl

    echo "== Configure and start services =="
    systemctl enable httpd && systemctl start httpd

    echo "== IPTables: Resetting FILTER table =="
    for CHAIN in 'INPUT' 'FORWARD' 'OUTPUT'; do
      iptables -P $CHAIN ACCEPT;
      iptables -F $CHAIN;
    done
    echo "== IPTables: Resetting MANGLE table =="
    for CHAIN in 'PREROUTING' 'INPUT' 'FORWARD' 'OUTPUT' 'POSTROUTING'; do
      iptables -t mangle -P $CHAIN ACCEPT;
      iptables -t mangle -F $CHAIN;
    done
    echo "== IPTables: Resetting NAT table =="
    for CHAIN in 'PREROUTING' 'INPUT' 'OUTPUT' 'POSTROUTING'; do
      iptables -t nat -P $CHAIN ACCEPT;
      iptables -t nat -F $CHAIN;
    done
    echo "== IPTables: Inserting Vagrant anti-lockout rule =="
    iptables -I INPUT  1 -p tcp --dport 22 -j ACCEPT -m comment --comment "Allow incoming SSH traffic"
    iptables -I OUTPUT 1 -p tcp --sport 22 -j ACCEPT -m comment --comment "Allow outgoing SSH traffic"
    echo "== IPTables: Inserting Allow Ping rule =="
    iptables -I INPUT  2 -p icmp --icmp-type echo-request -j ACCEPT -m comment --comment "Allow incoming Ping"
    iptables -I OUTPUT 2 -p icmp --icmp-type echo-reply -j ACCEPT -m state --state ESTABLISHED,RELATED -m comment --comment "Allow outgoing Ping"
    
    echo "== IPTables: Lockdown the Firewall"
    for CHAIN in 'INPUT' 'FORWARD' 'OUTPUT'; do
     iptables -A $CHAIN -j REJECT -m comment --comment "Reject all unmatched traffic";
    done
  SHELL
end
