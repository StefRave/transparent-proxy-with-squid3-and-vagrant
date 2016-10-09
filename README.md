# Transparent proxy in Vagrant

When you play with Vagrant boxes or Dockers, a lot of files will be downloaded
from internet again and again. For example, every time you run `apt-get update`,
it will download a set of files. The setup process will be much faster if these
files can be cached in local network. In this case, a transparent proxy can be
very helpful.

# How it works

Two Vagrant boxes will be created. Squid 3 is install in one box called squid,
and another box called client, is an example to demonstrate how to use the proxy.

The "squid" box has two NICs. The default NIC is used to access internet, while
a `private_network` with IP address 172.28.128.200 will be the gateway of clients.

After these boxes are up, you need to login the client box, change the default
gateway with command `change-gateway`, then most HTTP request will be cached.

Route table can be verified with this command, `netstat -r`:

```
root@client:~# netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         localhost       0.0.0.0         UG        0 0          0 eth1
10.0.2.0        *               255.255.255.0   U         0 0          0 eth0
172.28.128.0    *               255.255.255.0   U         0 0          0 eth1
```

To check if the proxy is working, initiate some HTTP requests from "client" box:

```
curl -v http://bing.com
curl -v http://bing.com
```

There will be following content in `/var/log/squid3/access.log` on "squid" box:

```
vagrant@squid:~$ sudo tail -f /var/log/squid3/access.log
1446885198.469    784 172.28.128.202 TCP_MISS/301 395 GET http://bing.com/ - HIER_DIRECT/204.79.197.200 -
1446885203.531      1 172.28.128.202 TCP_MEM_HIT/301 401 GET http://bing.com/ - HIER_NONE/- -
```

Enjoy it!

```

See the certificate returned for https connections
curl --insecure -v https://www.bing.com 2>&1 | awk 'BEGIN { cert=0 } /^\* Server certificate:/ { cert=1 } /^\*/ { if (cert) print }'

Get & restort with new config
sudo cp /vagrant/files/squid.conf /etc/squid/squid.conf
sudo squid -k reconfigure

Up & running?
netstat -an |grep 3130

sudo tail -f /var/log/squid/cache.log
sudo tail -f /var/log/squid/access.log
```

## Usefull SSL docs
http://wiki.squid-cache.org/Features/SslPeekAndSplice
http://marek.helion.pl/install/squid.html (not tried yet)


