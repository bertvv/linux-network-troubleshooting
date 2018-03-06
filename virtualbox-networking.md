# Troubleshooting VirtualBox networking

## Network interfaces

In a VirtualBox VM with EL7, the network interfaces are usually named as follows:

- Adapter 1: `enp0s3`
- Adapter 2: `enp0s8`
- Adapter 3: `enp0s9`
- Adapter 4: `enp0s10`

## NAT interface

The VirtualBox [NAT adapter](http://www.virtualbox.org/manual/ch06.html#network_nat)
is the default way of allowing your VM to access the Internet. However, it is normally not possible to access your VM from the host system over the network.

The network settings of your VM are predictable. Any VM with a NAT interface has:

- IP Address: 10.0.2.15/24
- Default gateway: 10.0.2.2
- Name server: 10.0.2.3 (potentially supplemented with the name servers assigned to your host system)

You can ping the gateway and name server from the VM, but pinging 10.0.2.15 from your host system will *never* work. Generally, if your host system has Internet, your VM also should have Internet. One possible exception is when you switch your host system from a wired to wireless connection, or vice versa, while your VM is running.

## Host-only interface

The VirtualBox [host-only adapter](http://www.virtualbox.org/manual/ch06.html#network_hostonly) allows you to access a VM over the network. A virtual Ethernet interface is created that is also visible on your host system. You can create multiple host-only adapters and configure network settings from the main window by opening File > Preferences > Network > Host-only Networks.

A host-only interface is named:

- On Linux/MacOS: `vboxnet0`, `vboxnet1`, ...
- On Windows: "VirtualBox Host-only Ethernet Interface". "VirtualBox Host-only Ethernet Interface #2", ...

The **default settings** for a host-only interface are:

- Adapter
    - IPv4 Address: 192.168.56.1 (= the IP-address assigned to your host system)
    - IPv4 Network Mask: 255.255.255.0
- DHCP-server: enabled
    - server address: 192.168.56.100 (= the IP address of a virtual DHCP server)
    - Server Mask: 255.255.255.0
    - Lower bound: 192.168.56.101
    - Upper bound: 192.168.56.254

If a VM is configured to get an IP address automatically (i.e. via DHCP), it will probably get 192.168.56.101 (if it's the first VM on this host-only network). If you want to assign static IP addresses to VMs on this network, take one from the range 2-99.

I suggest to leave the default host-only interface as is, and create new interfaces as needed. It may be needed to shut down all VMs and restart VirtualBox for configuration changes to come into effect, especially after creating a new interface.

### Checklist:

1. Check host-only network configuration: VirtualBox main window > File > Preferences > Network > Host-only Networks:
    - Are there any host-only adapters defined? If not, create one and check if the settings are as expected (see above)
    - Does the host system have an IP address in the form 192.168.NNN.1?
        - If the host system has a zeroconf/APIPA address (range 196.254.0.0/16), there is a problem with the Host-only network adapter settings: probably the value of the field "IPv4 Address" conflicts with an already assigned IP address on another interface.
    - Is the DHCP server activated and configured consistently with the IP address of the host system?
2. During the **Data Access Layer phase**, check the VM network adapters (VM Settings > Network)
    - Is the adapter enabled?
    - Is the cable connected (hidden setting under "advanced")
    - Is it attached to the correct host-only network?
3. During the **Network Layer phase**, in the VM, check the IP address of the adapter connected to the host-only network
    - Is it e.g. 192.168.56.101 (default host-only network, first IP address assigned by DHCP)
    - Is it e.g. 192.168.56.NN (with 2 <= NN <= 99, if you assigned a fixed IP address)
    - Does the host system have an IP address on that host-only network within the same IP network? Check the network mask as well!
4. Also during the **Network Layer phase**, ping the VM from your host system and vice versa. Remark that pinging the host system from the VM may not always work, e.g. if the firewall on the host system is configured to block ICMP traffic.
