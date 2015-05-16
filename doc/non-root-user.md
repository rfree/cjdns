Run cjdns as a non-root user
============================

There are two ways to run cjdns as a non-root user. You might need this if you
want to run cjdns on OpenVZ or start the network as a normal user.

Method 1: Setup the tun interface ahead of time (required for OpenVZ)
---------------------------------------------------------------------

If you are using an OpenVZ based VPS then you will need to use this as OpenVZ
does not permit persistent tunnels.

Create a cjdns user:

    sudo useradd cjdns

Create a new TUN device and give the cjdns user authority to access it:

    sudo /sbin/ip tuntap add mode tun user cjdns dev cjdroute0


### 4b-1: Setup the interface manually

Run those commands to prepare your TUN device:

    sudo /sbin/ip addr add <your ipv6 address>/8 dev cjdroute0
    sudo /sbin/ip link set mtu 1312 dev cjdroute0
    sudo /sbin/ip link set cjdroute0 up

These commands should be executed as root now every time the system restarts.
You might also want to place these commands in /etc/rc.local if your system
supports it.

#### Old versions of iproute2

If you see an error when running /sbin/ip, your version of iproute2 might be
old.

    sudo /sbin/ip tuntap add mode tun user cjdns
    Object "tuntap" is unknown, try "ip help".

The fix: for now grab a copy of a newer `ip` binary and copy it to your home
directory. Replacing the system binaries is not likely a good idea.

### 4b-2: Fire it up!

    sudo -u cjdns ./cjdroute < cjdroute.conf

To delete a tunnel, use this command:

    sudo /sbin/ip tuntap del mode tun <name of tunnel>

Method 2: Give the capabilities to create the tun device to the cjdroute executable
-----------------------------------------------------------------------------------

Run this command as root after building (and every time you rebuild cjdroute):

    setcap "cap_net_admin+eip cap_net_raw+eip" cjdroute


Grsecurity compatibility
-----------------------------------------------------------------------------------
The user building (and running?) cjdns seems to need exemption from TPE rules.
The /usr/bin/nodejs (or other patch of nodejs binary) seems to need pax flags, e.g. "mr"


Additional notes and examples
-----------------------------------------------------------------------------------
This seems to be needed (e.g. on Debian 7), run as root (at leasts once for given system):
ln -s /usr/bin/nodejs /usr/bin/node


Tun setup script (run as root after reboot). TODO(rfree)

#!/bin/bash 
# TODO(rfree) get the real internet card?
IPv6=rootip a |  grep inet6 | tail -n 1 | awk '{print $2 }'root
user="srv_cjdns"
# TODO(rfree) cjdroute0 device then needs to be changed in cjdroute.conf

echo "Setup network interface fo cjdns for user: $user with ip $IPv6"

set -x
/sbin/ip tuntap add mode tun user $user dev cjdroute0

/sbin/ip addr add $IPv6 dev cjdroute0
/sbin/ip link set mtu 1312 dev cjdroute0
/sbin/ip link set cjdroute0 up
set +x

ip a

