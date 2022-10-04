## Linux Networking

# Switching 

- Every computer system nbeeds an interface to connect to the external world. we can view the interface of our Linux system using:
`ifconfig` or `ip link` commands. This contains an `lo` which is usually a *loopback* interface, an `eth0` or something similar for the main interface which connects to the other networks ann internet as well.
- To enable our computer to reach the other system. we need a way to send the packets from an Gateway. a Gateway is the Door for our system. All the Gateway information is added in our system using `ip route` command, like so: `ip route add 192.168.2.5/24 via 192.168.2.1`.  
