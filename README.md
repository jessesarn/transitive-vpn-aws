# notes for AWS transitive VPN based on strongswan

Pre-requisites for EC2 instance:
- Launched in a VPC (new or existing)
- Use CentOS or RHEL AMI
- Source/Destination check has been disabled
- Install strongswan (i.e. sudo yum install strongswan)




### sysctl settings
`
$ cat /etc/sysctl.conf 
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
# Disable Source verification and send redirects to avoid Bogus traffic within OpenSWAN
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.eth0.rp_filter=0
net.ipv4.conf.lo.rp_filter=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.default.send_redirects=0
net.ipv4.conf.eth0.send_redirects=0
net.ipv4.conf.lo.send_redirects=0
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.eth0.accept_redirects=0
net.ipv4.conf.lo.accept_redirects=0
# Enable IP forwarding in order to route traffic through the instances
net.ipv4.ip_forward=1
`

### Sample ipsec.conf with AWS VPN Gateway
- left=xxx.xxx.xxx.xxx - Public IP of the Strongswan instance
- right=yyy.yyy.yyy.yyy - Outside IP of AWS VGW1
- mark=100 - Packets marked with this value will be tunneled
`
conn %default
	leftauth=psk
	rightauth=psk
	
    ike=aes256-sha256-modp2048s256,aes128-sha1-modp1024!
	ikelifetime=28800s
	aggressive=no
	esp=aes128-sha256-modp2048s256,aes128-sha1-modp1024!
	lifetime=3600s
	type=tunnel

	keyexchange=ikev1
	rekey=yes
	reauth=no
	dpdaction=restart
	dpddelay=10s
	dpdtimeout=30s
	closeaction=restart

	left=%defaultroute
	leftsubnet=0.0.0.0/0,::/0
	rightsubnet=0.0.0.0/0,::/0
	#leftupdown=/etc/strongswan/ipsec-vti.sh
	installpolicy=yes
	compress=no
	mobike=no

conn AWS-VPC-GW1
	# Customer Gateway
	left=xxx.xxx.xxx.xxx
	# VGW1 Outside IP
	right=yyy.yyy.yyy.yyy
	auto=start
	mark=100
	#reqid=1
`

### Sample ipsec.secrets with AWS VPN Gateway (Can include PSKs for other peers)
`
$ cat /etc/ipsec.secrets
# ipsec.secrets - strongSwan IPsec secrets file
xxx.xxx.xxx.xxx yyy.yyy.yyy.yyy : PSK "... preshared key for this set of peers ..."
xxx.xxx.xxx.xxx zzz.zzz.zzz.zzz : PSK "... preshared key for this set of peers ..."
...

`
