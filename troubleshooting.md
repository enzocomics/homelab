## Network Connectivity Issues 
- Check the list of network interfaces using `ip a`
- Check that the active interface is the one being bridged & the addresses are correct `nano /etc/network/interfaces` 
- Check if the DNS is correct in `nano /etc/resolv.conf`
- Reload the network configuration: `ifreload a`
