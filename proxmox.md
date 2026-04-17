# Post-install setup
- Run the post install script: `bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"` - [*source*](https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install&category=Proxmox+%26+Virtualization)
- Set up SMTP for datacenter notifications
- Installed Cloudflare's origin certificate onto the node

# Firewall setup
- Set up the datacenter firewall and open up any required ports (http, https, dns, etc) [*source*](https://pve.proxmox.com/wiki/Firewall). This is *required* or none of the lower-level firewalls will work
- Activate the node firewall
- Activate the firewall per virtual machine for:
  -  [Cloudflared](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/)
  -  [Adguard Home](https://github.com/AdguardTeam/AdGuardHome/wiki/Getting-Started)
  -  [Unifi Controller](https://help.ui.com/hc/en-us/articles/218506997-Required-Ports-Reference)
  - Be sure to check that the firewall is checked ON for each network interface
  - Each VM can also have a firewal, make sure that they are able to talk to each other (when required). Especially if you're using cloudflare tunnel, make sure the tunnel can actually reach the machines it needs to


# How I got my VLAN working with Proxmox
1. Create the VLAN on the network. For example, let's call it `homelab` with an ID of 2. Let's say the subnet is `192.168.2.0/24`
2. The port our node will be connecting to should have `homelab` tagged as its native VLAN
3. In Proxmox, create a new linux bridge and specify the bridge port that is connected to the port we just tagged
4. When creating a new virtual machine, the IP address must be static and set to an available address in the subnet. The gateway must also be set (it's probably `192.168.2.1`). DO NOT specify a VLAN ID during the install.
5. ???
6. Profit

# Checkmate
- Installed Golang by downloading the tar and SCPing it into the server
- Installed the server monitoring agent Capture [*source*](https://docs.checkmate.so/users-guide/server-monitoring-agent)
  - Run the capture server: `API_SECRET=<secret_token> GIN_MODE=release ./go/bin/capture &`
  - The `&` will let it run in the background so that exiting the shell won't terminate the process
  - If you want to kill the process in the future, look for the PID with `ps -C capture` and then kill it with `kill <PID>`
- Edited `vite.config.js` in the `/opt/checkmate/client` folder and added the app url to `preview.allowedHosts` [*source*](https://vite.dev/config/preview-options)

# Network Troubleshooting
- `lspci | grep Ethernet`: Show the bus id of all the network ports
- `ip a`: List all the current network interfaces
- `ethtool -i <interface>`: Shows more info about the network interface, including which bus
- `ifreload -a`: Refresh the network configuration if you've changed it in `/etc/network/interfaces`
- `systemctl restart networking`: or just reboot if it still doesn't refresh
- Don't forget to edit the `/etc/hosts` file if you end up changing the IP address of the host machine

# Cloning VMs
I've only tested this in Ubuntu but I imagine it's similar in other distributions: You can totally use a backup of a VM in a particular state (i.e. fresh install, one with all your SSH keys set up, etc) to create a new VM. But you'll have to change a few things:
- Give the machine a new ID:
  ```
  sudo rm -f /etc/machine-id
  sudo dbus-uuidgen --ensure=/etc/machine-id
  sudo rm /var/lib/dbus/machine-id
  sudo dbus-uuidgen --ensure
  sudo reboot
  ```
- Edit `/etc/netplan/00-installer-config.yaml` to change the IP & MAC address and then `netplan apply` to apply it
- Reference: [1](https://www.reddit.com/r/Ubuntu/comments/yqb1y0/cloning_ubuntu_linux_vms_identical_machine_id/) [2](https://linuxvox.com/blog/ubuntu-network-settings-command-line/)
