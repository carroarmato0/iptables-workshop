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
    yum install -y bind-utils httpd mod_ssl nmap-ncat

    echo "== Configure and start services =="
    systemctl enable httpd && systemctl start httpd
    echo -e "[Unit]\nDescription=hello\n[Service]\nRestart=always\nExecStart=/usr/bin/nc -l 127.0.0.1 1234 -k -c \'echo \"Woah! You Port-Forwarded to me!\"\'\n[Install]\nWantedBy=multi-user.target\n" > /etc/systemd/system/hello.service
    echo -e "[Unit]\nDescription=server-a\n[Service]\nRestart=always\nExecStart=/usr/bin/nc -l 127.0.0.1 8081 -k -c \'echo \"Hi! I am Server A!\"\'\n[Install]\nWantedBy=multi-user.target\n" > /etc/systemd/system/server-a.service
    echo -e "[Unit]\nDescription=server-b\n[Service]\nRestart=always\nExecStart=/usr/bin/nc -l 127.0.0.1 8082 -k -c \'echo \"Hi! I am Server B!\"\'\n[Install]\nWantedBy=multi-user.target\n" > /etc/systemd/system/server-b.service
    echo -e "[Unit]\nDescription=server-c\n[Service]\nRestart=always\nExecStart=/usr/bin/nc -l 127.0.0.1 8083 -k -c \'echo \"Hi! I am Server C!\"\'\n[Install]\nWantedBy=multi-user.target\n" > /etc/systemd/system/server-c.service
    systemctl daemon-reload
    systemctl enable hello    && systemctl start hello
    systemctl enable server-a && systemctl start server-a
    systemctl enable server-b && systemctl start server-b
    systemctl enable server-c && systemctl start server-c

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
