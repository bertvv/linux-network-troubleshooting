
# Transport layer

In the transport layer, we'll check whether the network service is actually running, what port it uses, and whether the firewall allows traffic on that port. An example for `httpd` is given, but this can be applied to other services.

## Service and port

- Is the service running? `sudo systemctl status httpd.service`
    - Expected output: `active (running)`
    - If the output contains: `inactive (dead)`, start the service, and if necessary, make sure it starts automatically on boot

        ```
        sudo systemctl start httpd.service
        sudo systemctl enable httpd.service
        ```

- What port is the service using? `sudo ss -tlnp` (list TCP (`-t`) server (`-l`) port numbers (`-n`) with the process behind them (`-p`). The `-p` option requires root, hence the `sudo`)
    - The expected output depends on the service and on how you configured it. The httpd service usually listens on port 80 (HTTP) or 443 (HTTPS), but the port number may have been set to a non-standard. Check `/etc/services` for standard port numbers for well-known network services.
    - Is the service listening on external interfaces? Often, the default configuration of a network service only listens on the loopback interface.

## Firewall setting

Does the firewall allow traffic on the service? `sudo firewall-cmd --list-all`.

```console
$ sudo firewall-cmd --list-all
[sudo] password for USER:
public (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client mdns samba-client ssh http https
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

Check the output for the following items:

- The network interface that the service listens on is listed
- The service name is listed.
    - The value should be one listed by `firewall-cmd --get-services`.
    - Remark that the service name for `firewalld` is *not necessarily* equal to the service name for `systemd`. E.g. BIND is called `named.service` by `systemd`, while it is referred to as `dns` by `firewalld`.
- If the service name is not present, the port numbers used by the service should be listed (e.g. when using a non-standard port, or a service not known by `firewalld`)
