# TL;DR checklist

The most important commands, usable in most situations. See the detailed guides for specific commands/things to check.

1. Physical/Network access layer
    - Check the cables
    - Check switch port LEDs
    - Is de box connected to the correct switch?
2. Internet layer:
    1. Check network settings:
        - `ip a`
        - `ip r` (+ ping default gw)
        - `cat /etc/resolv.conf` (+ `dig www.google.com @a.b.c.d +short`)
    2. LAN connectivity:
        - Ping between hosts, default GW
        - Check DNS availability: `nslookup`, `dig`, `getent ahosts`
4. Transport layer
    1. Is the service running? `sudo systemctl status SERVICE.service`
        - `sudo systemctl start SERVICE.service`
        - `sudo systemctl enable SERVICE.service`
    2. Does it use the correct ports/sockets? `sudo ss -tlnp`
    3. Does the firewall allow traffic to those ports? `sudo firewall-cmd --list-all`
        - `sudo firewall-cmd --add-service=SERVICE --permanent`
        - `sudo firewall-cmd --add-port=PORT/tcp --permanent`
        - `sudo systemctl restart firewalld`
5. Application layer
    - Check the logs: `sudo journalctl -f -u SERVICE.service`
    - Restart after each config file change: `sudo systemctl restart SERVICE.service`

