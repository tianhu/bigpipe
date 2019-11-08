![picture](tinc-icon.png)

# Step by step guide to setup tinc VPN

Tinc is a peer to peer VPN software to create a mesh network. (https://www.tinc-vpn.org)  
  
In this gudie, I will show you how to setup an Azure VPS act as a server named tiger and a local client named howard. 

## Server installation

1. `sudo apt install tinc`

2. `sudo vi /etc/sysctl.conf`

> uncommend: `net.ipv4.ip_forward=1`

3. `sudo sysctl -p`


4. `cd /etc/tinc`
5. `sudo mkdir -p tiger/hosts`
6. `cd tiger`
7. `sudo vi tinc.conf`

Write below content.  

> Name = tiger  
> AddressFamily = ipv4  
> Interface = tun0  

8. `sudo vi tinc-up`

Write below content.  

> #!/bin/sh  
> ip link set $INTERFACE up  
> ip addr add 192.168.60.1/24 dev $INTERFACE  
> iptables -A POSTROUTING -t nat -s 192.168.60.0/24 -j MASQUERADE -o eth0  

9. `sudo chmod +x tinc-up`
10. `sudo vi tinc-down`

Write below content.  

> #!/bin/sh  
> ip link set $INTERFACE down  
> iptables -D POSTROUTING -t nat -s 192.168.60.0/24 -j MASQUERADE -o eth0  

11. `sudo chmod +x tinc-down`
12. `cd hosts`
13. `sudo vi tiger`

Write below content. Replace `<server-public-ip>` with the server's public IP address.  

> Address = `<server-public-ip>`  
> Port = 443  
> Subnet = 0.0.0.0/0  

14. `sudo tincd -n tiger -K4096`
15. `sudo vi /etc/tinc/nets.boot`

Append a new line.  

> tiger  

16. `sudo systemctl enable tinc@tiger`
17. `sudo systemctl start tinc@tiger`


## Client installation

1. `sudo apt install tinc`
2. `cd /etc/tinc`
3. `sudo mkdir -p howard/hosts`
4. `cd howard`
5. `sudo vi tinc.conf`

Write below content.  

> Name = howard  
> AddressFamily = ipv4  
> Interface = tun0  
> ConnectTo = tiger  

6. `sudo vi tinc-up`

Write below content. Replace `<server-public-ip>` with the server's public IP address.  

> #!/bin/sh  
> ip link set $INTERFACE up  
> ip addr add 192.168.60.2/24 dev $INTERFACE  
> ip route add 192.168.60.254/24 dev $INTERFACE  
>   
> VPN_GATEWAY=192.168.60.1  
> REMOTEADDRESS=`<server-public-ip>`  
> ORIGINAL_GATEWAY=\`ip route show | grep ^default | cut -d ' ' -f 2-5\`  
>   
> ip route add $REMOTEADDRESS $ORIGINAL_GATEWAY  
> ip route add $VPN_GATEWAY dev $INTERFACE  
> ip route add 0.0.0.0/1 via $VPN_GATEWAY dev $INTERFACE  
> ip route add 128.0.0.0/1 via $VPN_GATEWAY dev $INTERFACE  

7. `sudo chmod +x tinc-up`
8. `sudo vi tinc-down`

Write below content. Replace `<server-public-ip>` with the server's public IP address.  

> #!/bin/sh  
> ip link set $INTERFACE down  
>   
> VPN_GATEWAY=192.168.60.1  
> ORIGINAL_GATEWAY=\`ip route show | grep ^default | cut -d ' ' -f 2-5\`  
> REMOTEADDRESS=`<server-public-ip>`  
>   
> ip route del $REMOTEADDRESS $ORIGINAL_GATEWAY  
> ip route del $VPN_GATEWAY dev $INTERFACE  
> ip route del 0.0.0.0/1 dev $INTERFACE  
> ip route del 128.0.0.0/1 dev $INTERFACE  

9. `sudo chmod +x tinc-down`
10. `cd hosts`
11. `sudo vi howard`

Write below content.  

> Subnet = 192.168.60.2/32  

12. `sudo tincd -n howard -K4096`

13. Exchange host files.  

> Copy the file `tiger` from the folder `/etc/tinc/tiger/hosts` on the server to the folder `/etc/tinc/howard/hosts` on the client.  
> Copy the file `howard` from the folder `/etc/tinc/howard/hosts` on the client to the folder `/etc/tinc/tiger/hosts` on the server.  

14. Launch tinc on demand with command `sudo tincd -n howard -D`

> Use command line `sudo pkill tincd` to stop.

15. Customize DNS

> You may want to specify an external DNS server to resolve some blocked domain names in some countries.  
>   
> ```
> sudo vi /etc/systemd/resolved.conf  
> ```
>  
> Set below settings.  
>   
>> [Resolve]  
>> DNS=8.8.8.8 8.8.4.4  
>> Domains=~.  
>   
> ```
> sudo systemctl restart systemd-resolved  
> ```
