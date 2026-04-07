# Network Setup - Legacy
- `ip a`: List network interfaces 
- `ethtool -i <interface_name>`: Check the PCI bus of `<interface_name>`
- `ethtool <interface name>`: Check the speed and if it's connected
- `/etc/network/interfaces`: file to change network interfaces
- `/etc/resolv.conf`: the value of `nameserver` should be the local network's DNS server
- `/etc/hosts`: match the hostname(s) to IP addresse(s)
- `/etc/hostname`: change the hostname (requires reboot)
- `ifup -a` / `ifreload -a` / `systemctl restart networking` to refresh after any changes

## VLAN Setup
In `/etc/network/interfaces`, each physical interface (i.e. `enp3s0`) will require a separate interface defined for each VLAN:
```
iface ens1 inet manual
# 1GBE port

iface ens1.2 inet manual
# 1GBE port tagged with VLAN ID 2
```
A bridge now can be defined with that interface:
```
auto vmbr1v2
iface vmbr1v2 inet static
    address 192.168.1.100/24
    bridge-ports ens1.2
    bridge-stp off
    bridge-fd 0
# Bridge tagged with VLAN 2
```
- Reference: [Proxmox Documentation](https://pve.proxmox.com/wiki/Network_Configuration#sysadmin_network_vlan)
# Network Setup - Cloud INIT + Network Manager
- `nmtui`: Terminal UI for network manager
- Set up a VLAN with an ID of `10` with `eth0` as the parent interface:
```
sudo nmcli connection add type vlan con-name vlan10 ifname eth0.10 dev eth0 id 10  
sudo nmcli connection modify vlan10 ipv4.method auto  
sudo nmcli connection up vlan10 
sudo nmcli connection modify eth0 ipv4.method disabled  
sudo nmcli connection down eth0
```
