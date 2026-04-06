# Network Setup
- `ip a`: List network interfaces 
- `ethtool -i <interface_name>`: Check the PCI bus of `<interface_name>`
- `ethtool <interface name>`: Check the speed and if it's connected
- `/etc/network/interfaces`: file to change network interfaces
- `/etc/resolv.conf`: the value of `nameserver` should be the local network's DNS server
- `/etc/hosts`: match the hostname(s) to IP addresse(s)
- `/etc/hostname`: change the hostname (requires reboot)
- `ifup -a` / `ifreload -a` / `systemctl restart networking` to refresh after any changes
