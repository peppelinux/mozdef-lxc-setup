Mozdef lxc setup
----------------
https://mozdef.readthedocs.io/en/latest/installation/manual/initial_setup.html#python-setup


Prepare the host and run the container
````
#!/bin/bash
CNT=mozdef
MOZDEF_USER=mozdef
MOZDEF_PATH=/opt/mozdef

apt install ifupdown lxc lxctl

lxc-create  -t download -n $CNT -- -d debian -r buster -a amd64
lxc-ls -f
# edit things and network parameters
nano /var/lib/lxc/$CNT/config

# notes for /var/lib/lxc/$CNT/config
# lxc.utsname = mozdef
# lxc Network configuration example
# lxc.network.type = veth
# lxc.network.flags = up
# lxc.network.link = lxc-br0
# lxc.network.name = mozdef0
# lxc.network.hwaddr = 00:FF:AA:02:03:01
# lxc.network.ipv4 = 10.0.3.10/24 10.0.3.255
# lxc.network.ipv4.gateway = 10.0.3.1

# host network example /etc/networking/interface
#auto lxc-br0
#iface lxc-br0 inet static
#      address 10.0.3.1
#      netmask 255.255.255.0
#      bridge_ports eth0
#      bridge_stp off
#      bridge_fd 2
#      bridge_maxwait 20

# host sys commands for poor-man-NAT
# sysctl -w net.ipv4.ip_forward=1
# iptables -t nat -A POSTROUTING -s 10.0.3.10 -o eth2 -j MASQUERADE

# run
lxc-start -n $CNT
lxc-attach -n $CNT
````

Inside the container
````
# Inside the container ...
CNT=mozdef
MOZDEF_USER=mozdef
MOZDEF_PATH=/opt/mozdef

apt update
apt install python3-pip python3-dev git \
            libcurl4-openssl-dev build-essential libssl-dev rsyslog

# which prefer?
#adduser $MOZDEF_USER -d MOZDEF_PATH
adduser --home $MOZDEF_PATH $MOZDEF_USER # passwd mozdef

mkdir -p $MOZDEF_PATH/envs
chown -R $MOZDEF_USER:$MOZDEF_USER $MOZDEF_PATH

pip3 install virtualenv
git clone https://github.com/mozilla/MozDef.git $MOZDEF_PATH/envs/mozdef

# activate the environment
source $MOZDEF_PATH/envs/python/bin/activate
cd $MOZDEF_PATH/envs/mozdef
virtualenv -p python3 $MOZDEF_PATH/envs/python
PYCURL_SSL_LIBRARY=nss pip3 install -r requirements.txt

# configure rsyslog
cp $MOZDEF_PATH/envs/mozdef/config/50-mozdef-filter.conf /etc/rsyslog.d/50-mozdef-filter.conf
service rsyslog restart

# supervisord
mkdir -p /var/log/mozdef/supervisord
chown -R $MOZDEF_USER:$MOZDEF_USER /var/log/mozdef
````

External services
-----------------
It’s recommended in a distributed environment, to have only 1 Alert Service node, 1 Web Service node, and N Ingest Service nodes.


## Elastic Search

MozDef currently only supports Elasticsearch version 6.8
````

````

## Kibana

````

````

## RabbitMQ

````

````

## MongoDB

````

````

## Nginx

````

````

.. then [Services](https://mozdef.readthedocs.io/en/latest/installation/manual/mozdef_services.html)
