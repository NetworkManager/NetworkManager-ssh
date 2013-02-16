# NetworkManager-ssh
Happy to introduce SSH VPN integration for NetworkManager.

It is still very much work in progress, so please do open me issues with bugs or future requests.
If there will be enough interest, I'll definitely continue developing it.
I still have enough cleanups I need to perform code wise.

I've forked the work of NetworkManager-ssh from NetworkManager-openvpn.

## Why?
Because we can!

SSH VPN can be used just anywhere!

## So what does it do?
Basically NetworkManager-ssh integrates OpenSSH tunnel capabilities with NetworkManager and provides you with the easiest of all VPNs, as OpenSSH lives on almost any *nix machine today.

## How does it work? SSH? VPN?
In order to open a tunnel OpenSSH VPN, all that you have to do is run:

	#!/bin/bash
	# This is the WAN IP/hostname of the remote machine
	REMOTE=EDIT_ME

	# Remote IP in the tunnel
	REMOTE_IP=192.168.0.1

	# Local IP in the tunnel
	LOCAL_IP=192.168.0.2

	# Netmask to set (on both sides)
	NETMASK=255.255.255.252

	# SSH port to use
	PORT=22

	# MTU for tunnel
	MTU=1500

	# Extra SSH options, these would give us some nice keep alive
	EXTRA_OPTS='-o ServerAliveInterval=10 -o TCPKeepAlive=yes'

	# Remote tunnel device (tun100)
	REMOTE_TUN=100

	# Local tunnel device (tun100)
	LOCAL_TUN=100

	ssh -f -v -o NumberOfPasswordPrompts=0 $EXTRA_OPTS \
		-w $LOCAL_TUN:$REMOTE_TUN \
		-l root -p $PORT $REMOTE \
		"ifconfig tun$REMOTE_TUN $REMOTE_IP netmask $NETMASK pointopoint $LOCAL_IP" && \
	ifconfig tun$LOCAL_TUN $LOCAL_IP netmask $NETMASK pointopoint $REMOTE_IP

That's actually an export file of a working SSH VPN configuration I have from NetworkManager.

This will create a tunnel of 192.168.0.1<->192.168.0.2 on tun100 on both machines. If forwarding is enabled on that SSH server, you'll get pass-through internet easy.

On the server, you'll need to enable in <i>/etc/ssh/sshd_config</i>:

	PermitTunnel=yes

## Compiling
### Fedora/CentOS
On Fedora things should be simple, after you clone the repository:

	./configure && make rpm

Enjoy your new RPM.

### Ubuntu/Debian
Building a .deb should be straight forward with (Tested on Ubuntu 12.10):

	./configure && make deb

Enjoy your new .deb.

## Running
Please edit <i>/etc/dbus-1/system.d/org.freedesktop.NetworkManager.conf</i> and add the line:

	<allow send_destination="org.freedesktop.NetworkManager.ssh"/>

Make sure your target host is known in <i>/root/.ssh/known_hosts</i>

If all went right, you should have a new VPN of type <i>SSH</i> when creating a new VPN.

## Limitations

### Authentication Types
Right now only ssh-agent authentication is supported, so you need to:

	eval `ssh-agent`
	ssh-add

NetworkManager-ssh probes for your ssh-agent (actualy at the moment any ssh-agent) and make the VPN connection authenticate with it.

It is very much a limitation and I'm looking for a more "elegant" way around it

### Known Hosts
If the destination host is not in your <i>known_hosts</i> file, things will also not work, unless you add in the extra options box:

	-o StrictHostKeyChecking=no

## Screenshots

Choosing a connection type:

<img src="https://raw.github.com/danfruehauf/NetworkManager-ssh/master/images/ConnectionType.png">

Main dialog:

<img src="https://raw.github.com/danfruehauf/NetworkManager-ssh/master/images/MainDialog.png">

Advanced dialog:

<img src="https://raw.github.com/danfruehauf/NetworkManager-ssh/master/images/AdvancedDialog.png">

