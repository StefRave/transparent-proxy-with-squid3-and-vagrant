# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box      = "ubuntu/trusty64"

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
    vb.memory = 512
    vb.cpus = 1
  end

  config.vm.define "squid" do |squid|
    squid.vm.hostname = "squid"
    squid.vm.network :private_network, ip: "172.28.128.200"

    squid.vm.provision "file", source: "files/squid.conf", destination: "/tmp/squid.conf"
    squid.vm.provision "file", source: "files/iptables.rules", destination: "/tmp/iptables.rules"
    squid.vm.provision "file", source: "files/restore-iptables",
      destination: "/tmp/restore-iptables"

    squid.vm.provision "shell", inline: <<-SHELL
      echo "deb http://ubuntu.diladele.com/ubuntu/ trusty main" > /etc/apt/sources.list.d/ubuntu.diladele.com.list

      apt-get update
      apt-get install -y --force=yes libecap3
      apt-get install -y --force-yes squid-common
      apt-get install -y --force-yes squid 
      apt-get install -y --force-yes squidclient

      mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
      mv /tmp/squid.conf /etc/squid/
      chown root:root /etc/squid/

      mkdir /etc/squid/ssl_cert/
      cp /vagrant/files/myCA.pem /etc/squid/ssl_cert/myCA.pem
      /usr/lib/squid/ssl_crtd -c -s /var/log/squid/ssl_db

      service squid restart

      mv /tmp/iptables.rules /etc/iptables.rules
      chown root:root /etc/iptables.rules
      mv /tmp/restore-iptables /etc/network/if-up.d/restore-iptables
      chown root:root /etc/network/if-up.d/restore-iptables
      iptables-restore < /etc/iptables.rules
    SHELL
  end

  config.vm.define "client" do |client|
    client.vm.hostname = "client"
    client.vm.network :private_network, ip: "172.28.128.202"

    client.vm.provision "file", source: "files/change-gateway", destination: "/tmp/change-gateway"
    client.vm.provision "shell", inline: <<-SHELL
      mv /tmp/change-gateway /usr/local/bin/change-gateway
      chmod +x /usr/local/bin/change-gateway
      echo "dns-nameservers 8.8.8.8" >> /etc/network/interfaces
    SHELL
  end
end
