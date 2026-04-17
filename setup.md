Start off with a fresh install of Proxmox (Debian).
- Run post-install scripts ([source](https://community-scripts.github.io/ProxmoxVE/scripts)): `bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"`
- Add public ssh key(s) into `/.ssh/authorized_keys`

## Install [IPMI Fan Control](https://github.com/Nammurg/ipmi_fan_control)
This is specific to our machine (Lenovo X3650 M5)
- `apt install ipmitool`
- `cd /usr/local/bin/`
- `curl -O https://raw.githubusercontent.com/Nammurg/ipmi_fan_control/refs/heads/main/lenovo_x3650_m5_ipmi_fan_control.sh`
- `chmod +x /usr/local/bin/lenovo_x3650_m5_ipmi_fan_control.sh` - makes the script executable

Let's make sure it runs when the system boots up.
- Create a systemd service file: `/etc/systemd/system/ipmi-fan-control.service`:
```service
[Unit]
Description=Automatically adjust the fan speed of the X3650 M5 based on the CPU temperature.

[Service]
# Make sure the script only runs once
Type=oneshot
ExecStart=/usr/local/bin/lenovo_x3650_m5_ipmi_fan_control.sh

[Install]
WantedBy=default.target
```
- `chmod 664 /etc/systemd/system/ipmi-fan-control.service` - Make sure the permissions are correct
- `systemctl daemon-reload` - tell system to read the service file
- `systemctl enable ipmi-fan-control.service` - enable it
- `systemctl is-enabled ipmi-fan-control.servic` - check if it's enabled

## **Install Fail2Ban**
- `apt install fail2ban`
- Create local config files: `cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local` and `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
- Edit `jail.local` and set the following:
```conf
[default]
backend = systemd
destemail = (your email)
sender = (email of the server)

[sshd]
enabled = true
```
- `systemctl enable fail2ban` and `systemctl start fail2ban`
- Check if it's running: `service --status-all` or `service fail2ban status`

## **Install Speedtest**
(Because I wanted to know if my 2.5gb ethernet card was working)
- `curl -OL https://github.com/taganaka/SpeedTest/archive/refs/heads/master.zip` - the `-l` option makes sure the github link redirect is followed
- Install dependencies: `apt install zip build-essential libcurl4-openssl-dev libxml2-dev libssl-dev cmake`
- Open the archive: `unzip master.zip`
- Navigate to the folder `cd SpeedTest-master` and then run `cmake -DCMAKE_BUILD_TYPE=Release .` and `make install`
- Run the program: `SpeedTest`


## IMM2 SMTP
- Must have a DNS server set
- SMTP reverse-path MUST be the same as the username
