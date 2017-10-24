# Troubleshooting network services in Linux

In this guide, I try to compile everything I know about troubleshooting network services on Linux systems (e.g. Apache, BIND, Samba, etc.).

## Assumptions

- The init system is `systemd`
- The primary target Linux distribution is Enterprise Linux 7 (RedHat Enterprise Linux/CentOS) and Fedora, but the principles and most commands are also valid on other Linux (and UNIX) distributions. YMMV.

## A bottom-up approach

When troubleshooting, it is essential to be systematic and thorough. As a model for a systematic troubleshooting approach, use the TCP/IP protocol stack, and follow the layers from bottom to top.

| Layer          | Protocols                | Keywords              |
| :---           | :---                     | :---                  |
| Application    | HTTP, DNS, SMB, FTP, ... |                       |
| Transport      | TCP, UDP                 | sockets, port numbers |
| Internet       | IP, ICMP                 | routing, IP address   |
| Network access | Ethernet                 | switch, MAC address   |
| Physical       |                          | cables                |

## General guidelines

A few best practices when setting up and troubleshooting network services:

* Compile a checklist and keep it **up-to-date** as you gain experience
* Be **systematic** and follow the same steps in the "correct" order
* Be **thorough**, don't skip steps (e.g checking the cables), unless it's proven that everything at that level works as expected.
* Work in **small steps** and verify every step. Verify every change you make.
* Don't assume. **Test**.
* Keep a **backup** copy of the original configuration, and the latest "working" version.
* Always **validate the syntax** of config files before applying them
* Know what **logs** to look at
* Open a separate terminal that shows the **logs in realtime** (e.g. `journalctl -f`)
* **Automate** your tests (Shell script, [BATS](https://github.com/sstephenson/bats) test script, Ansible playbook, Serverspec, etc.)
* **Error messages** usually give a clue of where to look and may provide a shortcut in the troubleshooting process. Look for the error message in the logs or returned by client software and know how to interpret them.

## Contents

1. The Physical and Network Access layer
2. The Internet layer
3. The Transport layer
4. The Application layer

## TL;DR checklist

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

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See the [license](LICENSE.txt) for details

![CC-BY-SA-4.0](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)

## Contributors

Issues, feature requests, ideas, suggestions, etc. are appreciated and can be posted in the Issues section.

Pull requests are also very welcome. Please create a topic branch for your proposed changes. If you don’t, this will create conflicts in your fork when you synchronise changes after the merge. Don’t hesitate to add yourself to the contributor list below in your PR!

- [Bert Van Vreckem](https://github.com/bertvv/) (maintainer)
