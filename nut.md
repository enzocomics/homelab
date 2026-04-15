## Nut Client
```
apt install nut-client nut-server -y
```
```
systemctl disable --now nut-server
```
Edit `/etc/nut/nutc.conf`:
```
MODE=netclient
```
Check connectivity with server:
```
upsc ups-name@server
```
Configure client monitoring in `/etc/nut/upsmon.conf`:
```
RUN_AS_USER root
MONITOR ups-name@server 1 upsuser upspassword secondary
NOTIFYCMD /usr/sbin/upssched
NOTIFYFLAG ONBATT EXEC
```
Start service:
```
systemctl restart nut-client
systemctl status nut-client
systemctl enable --now nut-client
```
Shut down on power loss in `/etc/nut/upssched.conf`:
```
CMDSCRIPT /usr/bin/upssched-cmd
PIPEFN /run/nut/upssched/upssched.pipe
LOCKFN /run/nut/upssched/upssched.lock
# Execute shutdown as soon as UPS goes on battery
AT ONBATT * EXECUTE shutdown_onbatt
```
Edit `/usr/bin/upssched-cmd`:
```
#!/usr/bin/env bash
case "$1" in
  shutdown_onbatt)
    logger -t nut-monitor "Received ONBATT from server: shutting down"
    /sbin/shutdown -h now
    ;;
  *)
    logger -t nut-monitor "upssched-cmd: unknown argument: $1"
    ;;
esac
```
Make it executable:
```
chmod +x /usr/bin/upssched-cmd
```
Restart:
```
systemctl restart nut-client

systemctl status nut-client
```
Test:
```
/usr/bin/upssched-cmd shutdown_onbatt
```


### References:
- https://blog.juanje.net/en/posts/nut-monitoring/
- https://technotim.com/posts/NUT-server-guide/#linux-nut-client-remote
