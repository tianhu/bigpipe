## Server

`sudo apt install tinc`

`sudo vi /etc/sysctl.conf`

> uncommend: net.ipv4.ip_forward=1

`sudo sysctl -p`


`cd /etc/tinc`
`sudo mkdir -p tiger/hosts`
`cd tiger`
`sudo vi tinc.conf`

> Name = tiger  
> AddressFamily = ipv4
> Interface = tun0

`sudo vi tinc-up`

> #!/bin/sh
> ip link set $INTERFACE up
> ip addr add 192.168.60.1/24 dev $INTERFACE
> ip route add 192.168.60.0/24 dev $INTERFACE
> iptables -A POSTROUTING -t nat -s 192.168.60.0/24 -j MASQUERADE -o eth0

`sudo chmod +x tinc-up`
`sudo vi tinc-down`

> #!/bin/sh
> ip link set $INTERFACE down
> iptables -D POSTROUTING -t nat -s 192.168.60.0/24 -j MASQUERADE -o eth0

`sudo chmod +x tinc-down`
`cd hosts`
`sudo vi tiger`

> Address = <server-public-ip>
> Port = 443
> Subnet = 0.0.0.0/0

`sudo tincd -n tiger -K4096`

`sudo vi /etc/tinc/nets.boot`

> Append a new line: tiger

`sudo systemctl enable tinc@tiger`
`sudo systemctl start tinc@tiger`


## Client

`sudo apt install tinc`
`cd /etc/tinc`
`sudo mkdir -p howard/hosts`
`cd howard`
`sudo vi tinc.conf`

> Name = howard
> AddressFamily = ipv4
> Interface = tun0
> ConnectTo = tiger

`sudo vi tinc-up`

> #!/bin/sh
> ip link set $INTERFACE up
> ip addr add 192.168.60.2/24 dev $INTERFACE
> ip route add 192.168.60.0/24 dev $INTERFACE

`sudo chmod +x tinc-up`
`sudo vi tinc-down`

> #!/bin/sh
> ip link set $INTERFACE down

`sudo chmod +x tinc-down`
`cd hosts`
`sudo vi howard`

> Subnet = 192.168.60.2/32

`sudo tincd -n howard -K4096`


`sudo tincd -n howard -D`


## exchange host files
Copy howard to tiger machine, copy tiger to howard machine.
