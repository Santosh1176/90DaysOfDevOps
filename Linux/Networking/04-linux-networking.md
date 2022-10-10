# Linux Neyworking 101

With Computers talking to each other over a vast infrastructure of cables and airwaves. In this chapter, I am exploring how this is accomplished, but with a prespective of a Linux user.

Every Linux box needs an interface to connect to other hardware systems. On most of the hardwares, this interface is defined by TCP/IP through which the hardware is accessed. TCP/IP offers a set of operations which basically deals with sending and receiving packets.

On Linux such interface is termed as `eth0`, `eith1`, for people who have docker installed on their Linus system might have also seen `docker0`. These datails can be assessd on any Linux system by passing `ifconfig` on your command prompt. The output would be similar to:

```bash
✗ ifconfig
br-4d351f0a6297: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.0.1  netmask 255.255.0.0  broadcast 172.20.255.255
        inet6 fe80::42:32ff:fe97:d00b  prefixlen 64  scopeid 0x20<link>
        inet6 fc00:f853:ccd:e793::1  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::1  prefixlen 64  scopeid 0x20<link>
        ether 02:42:32:97:d0:0b  txqueuelen 0  (Ethernet)
        RX packets 103  bytes 6524 (6.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 148  bytes 22878 (22.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:dc:77:b2:91  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.5  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::3c80:351f:9cb6:af13  prefixlen 64  scopeid 0x20<link>
        ether a0:48:1c:98:52:72  txqueuelen 1000  (Ethernet)
        RX packets 18889  bytes 11823140 (11.8 MB)
        RX errors 0  dropped 257  overruns 0  frame 0
        TX packets 16530  bytes 5254064 (5.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 20  memory 0xf7c00000-f7c20000
```

AS you can see, the systems default GAteway is set to `192.168.1.1`. This implies all the traffic from SYSTEM-A will flow through this Gateway. Similarly, if any host from SYSTEM-B wants to connect to SYSTEM-A. We need to add the Routing information in that host. This can be acheived by applying the following command in SYSTEM-B's host:

`ip route add 192.168.2.5/24 via 192.168.2.1`. Remember to add teh Routing tables in all the systems of an internal network. The details of the same can be found by command `ip route`. Now, how is our system going to connect to the Internet? Notice the **Default** in the Destination column of the above Routing table. This tells our system to route all the traffic outside our network from the IP address set to this i.e `192.168.1.1`. 

*ps: In some Linux machines, you might see `0.0.0.0` instead of `Default`. Its one and the same.**

In any of the Linux system, whenever there are two interfaces connected to their respective networks. The Linux kernel, fro security reasons does't forward packets from one interface to the other (for example, from `en01` to `eno2`). If we explictly need to forward packets from differnt interfaces within our system, we need to define this setting in `/proc/sys/net/ipv4/ip_forword` file:
```bash
✗ cat /proc/sys/net/ipv4/ip_forward
  0
```
By default the configuration for **IP forwarding** is set to zero, to enable IP forwarding set its value to 2 using this command:
` echo 1 > /proc/sys/net/ipv4/ip_forward` to persist this setting across reboots, we also need to chage this setting in `/etc/sysctl.conf` by modifing the following entry:

** This setting might compromise your security. Enable this if you need packet forwarding from one interface to another**

```bash
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```


# DNS

Binding a Name to an Ip address. This can be achived by adding an entry in `/etc/hosts` file.
like so
```bash
cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       santoshdts
192.168.1.2     santosh
```
now a system within my own network with its IP set as `192.168.1.3` as example can pin me as `ping santosh`. Whenever we ping `www.google.com`,  or try to `ssh` into it, our Linux machine first looks into this `etc/host` file to check if there is any entry for `www.google.com`. If the entry was fount it forwards the packets to the connected IP address, if not It will forward the packets to and external DNS Nameserver through the Gateway we've configured.(This seting of first looking at host file then into Nameserver is configured in `/etc/nsswitch.conf`) The configuration of the DNS Nameserver can be found on:
```bash
> cat /etc/resolv.conf
<snip>
nameserver 127.0.0.53
options edns0 trust-ad
search .
```
This process is known as Name resolution.

# CoreDNS



# Network Namespace on Linux


# Resources:

- [Linux IP command cheatsheet by Redhat](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf)
- [Container Networking Is Simple!](https://iximiuz.com/en/posts/container-networking-is-simple/) by [Ivan Velichko](https://twitter.com/iximiuz)