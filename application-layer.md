# Application layer

The specific things on this layer mostly depend on the service in question. Apache, BIND, Vsftpd, Postfix, etc. all have quite different architecture and configuration.

However, there are a few general things that should be checked:

- the log files (for error messages that indicate the root cause of problems);
- whether the application is configured correctly;
- whether the service is available to clients, and responds correctly to requests/queries.

## Log files

Check the log, either using `journalctl`, or by looking at the log file in `/var/log`. The former is standard, the latter may be needed for services that keep a log file not managed/recognized by `systemd-journald`.

Open a separate terminal with the relevant logs opened and watching for changes (`-f` option). E.g.:

- `sudo journalctl -f -u httpd.service`
- `sudo tail -f /var/log/httpd/error_log`

## Configuration files

Check the configuration file, somewhere in `/etc/`, e.g. `/etc/httpd/httpd.conf`. First, create a backup of the current configuration file(s), and if possible the default one (as created when installing the service)

- Validate the syntax of the configuration file. Most services have a command that does this.
    - e.g. for `httpd`: `apachectl configtest`
- Check the configuration file for errors
- After making changes, restart the service, e.g.
    - `sudo systemctl restart httpd.service`

## Availability

You can start checking availability on the loopback interface of the server itself, but it is important to repeat this from another host. The loopback interface is not firewalled, while physical network interfaces are.

- Do a port scan from another host on the lan, e.g.
    - `sudo nmap -sS -p 80,443 HOST` (perform a TCP SYN scan on port 80 and 443)
- Use a test tool or client software to check availability of the service, e.g.
    - `wget http://HOST/`, `wget https://HOST/`
    - `curl http://HOST/`, `curl https://HOST/`
- Sometimes it's useful to set up a packet sniffer like Wireshark or Tcpdump to investigate network traffic between clients and the service.
