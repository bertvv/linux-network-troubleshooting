# Introduction

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

And finally

* **DO NOT `ping www.google.com`!!!**. Too many conditions need to be fulfilled for this command to succeed, so it is never a good troubleshooting command.
