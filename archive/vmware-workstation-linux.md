# install

yum -y install libXtst-devel libXrender-devel kernel-devel
yum -y install xinetd

./vmware-workstation-full-10.0.0.1295980-x64.bundle --custom
shared workstations: /var/www/vmware
port: 8443

# register

/usr/lib/vmware/bin/vmware-vmx --new-sn 5F29M-48312-8ZDF9-A8A5K-2AM0Z

# uninstall

./vmware-workstation-full-10.0.0.1295980-x64.bundle --uninstall-product vmware-workstation

# start

service vmware start

# stop VMX

ps axuw | grep vmware-vmx

# show vmware workstation version

vmware -v

# register newly downloaded vm

# copy to shared vm's folder: /var/www/vmware

vmrun -T server -h https://127.0.0.1:8443 -u root -p <root-password> register "\[standard] Windows XP Home Edition SP3/Windows XP Home Edition SP3.vmx"

Possibly will be needed to set video card settings to: "1 monitor" if related errors occured

# unregister vm

vmrun -T server -h https://127.0.0.1:8443 -u root -p <root-password> unregister "\[standard] Windows XP Home Edition SP3/Windows XP Home Edition SP3.vmx"

or via vmware workstation interface

---

Connect from Windows
Используем установленный vmware как клиент
подключаемся к порту, который указали при инсталле, mediaspider-vpn:8443
используем логин пароль юниксового юзера, например рута

use vmrun to manage workstation

# setup vmnet8 NAT network

# /etc/vmware/networking

answer VNET_8_HOSTONLY_SUBNET 192.168.175.0

# /etc/vmware/vmnet8/nat/nat.conf

ip = 192.168.175.2

# setup ports forwarding

# /etc/sysctl.conf

net.ipv4.ip_forward = 1

# enable net.ipv4.ip_forward = 1 for current session

sysctl -w net.ipv4.ip_forward=1

# manually configure ports forwarding in iptables script

# or configure ports forwarding in /etc/vmware/vmnet8/nat/nat.conf, in this case need to open connection to desired TCP port in iptables

# in virtual machine you need to specify static IP in range 192.168.175.10 - 192.168.175.255

# setup RDP on windows virtual machine

# in windows firewall allow RDP connections from public networks, if not enabled yet
