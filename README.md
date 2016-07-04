# ecmp-vagrant
Vagrant environment with ansible provisiong to build a test ECMP BGP environment

My goal was to set up the siplest possible BGP ECMP environment using Vagrant and Linux with whatever routing software I could get working.

I settled on the following configuration:

r1: quagga bgp AS 65000, peers AS 65001 (s1) AS 65002 (s2) AS 65003 (s3)
s1, s2, s3: bird bgp with peer 65000, all advertising anycast address 172.16.100.100

        +-----------------+
        |      client     |
        | 192.168.100.100 |
        +-------+---------+
                |
                |
            +---+----+
            |   r1   |
            |AS 65000|
            +--------+
        +------------------+
        |        |         |
    +---+--+  +--+---+  +--+---+
    |  s1  |  |  s2  |  |  s3  |
    |65001 |  | 65002|  |65003 |
    +------+  +------+  +------+
        ((( 172.16.100.100 ))

## Working
- You can communicate between 192.16.100.100 and 172.16.100.100:
  
`$ curl -s 172.16.100.100`
  
```This is node 1
socket info:{"address":"172.16.100.100","family":"IPv4","port":80}```

- You can kill a server's bird bgp instance and a new server will start routing the traffic

```
vagrant@r1:~$ ip route
default via 10.0.2.2 dev eth0
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
10.1.1.0/24 dev eth1  proto kernel  scope link  src 10.1.1.254
10.1.2.0/24 dev eth2  proto kernel  scope link  src 10.1.2.254
10.1.3.0/24 dev eth3  proto kernel  scope link  src 10.1.3.254
172.16.100.0/24 via 10.1.1.2 dev eth1  proto zebra           <<<<<<<<<< s1 is handling the route
192.168.100.0/24 dev eth4  proto kernel  scope link  src 192.168.100.254
```

`vagrant@s1:~$ sudo systemctl stop bird.service`

```
vagrant@r1:~$ ip route
default via 10.0.2.2 dev eth0
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
10.1.1.0/24 dev eth1  proto kernel  scope link  src 10.1.1.254
10.1.2.0/24 dev eth2  proto kernel  scope link  src 10.1.2.254
10.1.3.0/24 dev eth3  proto kernel  scope link  src 10.1.3.254
172.16.100.0/24 via 10.1.3.2 dev eth1  proto zebra                 <<<<<<<<<<< here's what changed
192.168.100.0/24 dev eth4  proto kernel  scope link  src 192.168.100.254
```

## Not Working

- There is no loadbalancing to the server instances.  This is either because:
 - The default debian jessie image I am using does not have multipath compiled in linux
 - the quagga binary package of zebra is not compiled with multipath support
 
So my current goal is to work until I get a routing config in linus that looks like this:

```
vagrant@r1:~$ ip route
default via 10.0.2.2 dev eth0
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
10.1.1.0/24 dev eth1  proto kernel  scope link  src 10.1.1.254
10.1.2.0/24 dev eth2  proto kernel  scope link  src 10.1.2.254
10.1.3.0/24 dev eth3  proto kernel  scope link  src 10.1.3.254
172.16.100.0/24 via 10.1.1.2 dev eth1  proto zebra
172.16.100.0/24 via 10.1.2.2 dev eth2  proto zebra
172.16.100.0/24 via 10.1.3.2 dev eth3  proto zebra
192.168.100.0/24 dev eth4  proto kernel  scope link  src 192.168.100.254
```
