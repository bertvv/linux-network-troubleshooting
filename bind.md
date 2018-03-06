# Troubleshooting BIND

This is not a BIND manual, so if you are not familiar with the configuration/zone file syntax, first read up on the subject. Recommended resources:

- [DNS for rocket scientists](http://www.zytrax.com/books/dns/)
- [DNS Servers](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/ch-dns_servers) in Red Hat Enterprise Linux Networking Guide

## General checks

- Log files: `sudo journalctl -f -u named.service`
    - Normally, BIND does not log incoming DNS requests. For debugging purposes, it may be useful to enable this. You could change the config file (to enable query logging permanently), or issue the command `rndc querylog on` (enable until next service restart).
- Configuration file syntax validation:
    - Main configuration file: `sudo named-checkconf /etc/named.conf`
    - Zone files: `sudo named-checkzone ZONE ZONE_FILE`, e.g.
        - `sudo named-checkzone cynalco.com /var/named/cynalco.com` (forward lookup zone)
        - `sudo named-checkzone 2.0.192.in-addr.arpa /var/named/2.0.192.in-addr.arpa` (reverse lookup zone)
- Availability: use `dig` (preferred) or `nslookup` to query the DNS server, e.g.:
    - `dig @DNS_SERVER_IP HOSTNAME`
    - `nslookup HOSTNAME DNS_SERVER_IP`

## Using `dig` and `nslookup`

Both commands can be installed with `yum install bind-utils`, if necessary. `dig` has more options and gives more detailed output, so it is preferred over `nslookup`. `nslookup` has the advantage that it is also available on Windows.

### `nslookup`

```console
$ nslookup www.hogent.be
Server:		195.130.131.1
Address:	195.130.131.1\#53

Non-authoritative answer:
Name:	www.hogent.be
Address: 178.62.144.90
```

In this example, the IP address for host `www.hogent.be` is queried. We get a response from DNS server 192.130.131.1. The output specifies that the answer is non-authoritative, i.e. the DNS server that provided the response is not the authoritative DNS server for the domain *hogent.be*.

When troubleshooting a DNS service, it is important to send the DNS query to the specific host running BIND. Therefore, specify its IP address on as an argument:

```console
$ nslookup www.hogent.be 8.8.8.8
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	www.hogent.be
Address: 178.62.144.90
```

### `dig`

The output of `dig` looks like this:

```console
$ dig www.hogent.be

; <<>> DiG 9.10.5-P2-RedHat-9.10.5-2.P2.fc25 <<>> www.hogent.be
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23001
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.hogent.be.			IN	A

;; ANSWER SECTION:
www.hogent.be.		2796	IN	A	178.62.144.90

;; Query time: 11 msec
;; SERVER: 195.130.131.1#53(195.130.131.1)
;; WHEN: Tue Sep 26 00:45:51 CEST 2017
;; MSG SIZE  rcvd: 58
```

The output syntax is compatible with that of zone files. Here, we see that there is an A record for www.hogent.be, viz. 178.62.144.90. All lines beginning with semicolons `;` are comments in the zone file syntax. The comments can be ommitted with option `+short`:

```console
$ dig +short www.hogent.be
178.62.144.90
```

Specify the DNS server to be queried as a command line argument, prefixed with `@`:

```console
$ dig +short @8.8.8.8 www.hogent.be
178.62.144.90
```

Other types of queries:

- who is the authoritative name server for domain *hogent.be*?

    ```console
    $ dig +short NS hogent.be
    ns2.belnet.be.
    ens2.hogent.be.
    ns1.belnet.be.
    ens1.hogent.be.
    ```

- what is the IPv6 address of *download.fedoraproject.org*?

    ```console
    $ dig +short AAAA download.fedoraproject.org
    wildcard.fedoraproject.org.
    2001:4178:2:1269::fed2
    2610:28:3090:3001:dead:beef:cafe:fed3
    2605:bc80:3010:600:dead:beef:cafe:fed9
    ```

- reverse lookup (only works if the queried DNS server has a PTR record for the specified IP address):

    ```console
    $ dig +short -x 195.130.131.1
    asse.dnscache02.telenet-ops.be.
    ```

## Common mistakes

The concept of a zone file is relatively simple: a text file containing hostname-to-IP mappings. However, the syntax is a bit arcane and small mistakes can have severe consequences. This section lists a few common mistakes.

### End fully qualified domain names with a dot

A fully qualified domain name should always end on a dot, e.g.

```
www.hogent.be.		2796	IN	A	178.62.144.90
```

If the dot is omitted, the value of `$ORIGIN` (specified at the beginning of the zone file) is appended. In a zone file for domain *hogent.be*, `www.hogent.be` (without the dot) would be interpreted as `www.hogent.be.hogent.be.`

### Reverse lookup domain notation

The notation for reverse lookup domains, specified by an IP subnet, is kind of weird. Take the IP network 192.0.2.0/24 as an example:

- Dotted quad notation is used, but the quads are written in reverse order: 0.2.0.192
- The host part of the network address is omitted: 2.0.192
- `in-addr.arpa.` (remark the dot at the end!) is added: `2.0.192.in-addr.arpa.`

The reverse lookup domain for 10.0.0.0/8 would be `10.in-addr.arpa.`, for 172.16.0.0/16 you would get `16.172.in-addr.arpa.`

### The serial

The SOA (start of authority) resource record always specifies a *serial*, an indicator of the version/revision of the zone file. You should always increase the serial after any change in the zone file. If you don't, zone transfers to slave servers will not be performed.

### Error "ignoring out-of-zone data"

### Running queries locally works, bot not remotely

If the BIND service responds to DNS queries when run on the server itself, but refuses queries from clients, check the main config file `/etc/named.conf`. It's probably configured to only respond to local requests. Specifically, check the options `listen-on`, `listen-on-v6` and `allow-query`. The default settings (at least on RHEL/CentOS) are to only listen to the loopback interface and only respond to queries from localhost:


```
options {
  listen-on port 53 { 127.0.0.1; };
  listen-on-v6 port 53 { ::1; };
  allow-query { localhost; };

  // ...
}
```

Changing these settings to `any;` will allow any host to send queries to this BIND service.
