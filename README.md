# Section 4

## Docker instructions

### 27.- None Network

```
docker run -d --net none busybox sleep 1000
docker exec -it <imageId> /bin/ash
ping 8.8.8.8 # Inside the container, error it does not have internet access
ifconfig # Loopback interface
```

* Provides the maximum level of network protection
* Not a good choice if network of Internet connection is required.
* Suites well where the container require the maximum level of network security and network access
is not necessary.

### 28.- Bridge Network

```
docker network ls
docker network inspect bridge
docker run -d --name container_1 busybox sleep 100
docker exec -it container_1 ifconfig # loopback and eth0

(venv) ➜  section4 git:(master) ✗ docker exec -it container_1 ifconfig                                                     
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:11 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:898 (898.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker run -d --name container_2 busybox sleep 1000
docker exec -it container_2 ifconfig

(venv) ➜  section4 git:(master) ✗ docker exec -it container_2 ifconfig              
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03  
          inet addr:172.17.0.3  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:508 (508.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)


docker exec -it container_1 ping 8.8.8.8 => OK
docker exec -it container_2 ping 8.8.8.8 => OK
```

```
docker network create --driver bridge my_bridge_network
docker network inspect my_bridge_network
docker run -d --name container_3 --net my_bridge_network busybox sleep 1000
docker exec -it container_3 ifconfig
docker inspect container_1 # to get the IP ("IPAddress": "172.17.0.2")
docker exec -it container_3 ping 172.17.0.2 => ERROR
docker network connect bridge container_3
docker exec -it container_3 ifconfig

➜  section4 git:(master) ✗ docker exec -it container_3 ifconfig       
eth0      Link encap:Ethernet  HWaddr 02:42:AC:13:00:02  
          inet addr:172.19.0.2  Bcast:172.19.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:23 errors:0 dropped:0 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1758 (1.7 KiB)  TX bytes:1806 (1.7 KiB)

eth1      Link encap:Ethernet  HWaddr 02:42:AC:11:00:04  
          inet addr:172.17.0.4  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:578 (578.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker exec -it container_3 ping 172.17.0.2 => OK
docker network disconnect bridge container_3
docker exec -it container_3 ifconfig

➜  section4 git:(master) ✗ docker exec -it container_3 ifconfig        
eth0      Link encap:Ethernet  HWaddr 02:42:AC:13:00:02  
          inet addr:172.19.0.2  Bcast:172.19.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:25 errors:0 dropped:0 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1898 (1.8 KiB)  TX bytes:1806 (1.7 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

#### Bridge Network

* In a bridge network, containers have access to two network interfaces.
    * A loopback interface
    * A private interface
* All containers in the same bridge network can communicate with each other.
* Containers from different bridge networks can't connect with each other by default.
* Reduces the level of network isolation in favor of better outside connectivity.
* Most suitable where you want to set up a relatively small network on a single host.

### 29.- D3: Host Network and Overlay Network

```
docker run -d --name container_4 --net host busybox sleep 1000
docker exec -it container_4 ifconfig

➜  section4 git:(master) ✗ docker exec -it container_4 ifconfig
br-4589fa4030e4 Link encap:Ethernet  HWaddr 02:42:DD:E0:F6:68  
          inet addr:172.19.0.1  Bcast:172.19.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:ddff:fee0:f668/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:19 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1540 (1.5 KiB)  TX bytes:1040 (1.0 KiB)

br-b4f1f90ab0b8 Link encap:Ethernet  HWaddr 02:42:E7:C6:7C:DE  
          inet addr:172.18.0.1  Bcast:172.18.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:e7ff:fec6:7cde/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:55 errors:0 dropped:0 overruns:0 frame:0
          TX packets:75 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:6026 (5.8 KiB)  TX bytes:9372 (9.1 KiB)

docker0   Link encap:Ethernet  HWaddr 02:42:B0:90:0F:26  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:b0ff:fe90:f26/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:1856 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2760 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:120735 (117.9 KiB)  TX bytes:2530386 (2.4 MiB)

eth0      Link encap:Ethernet  HWaddr 02:50:00:00:00:01  
          inet addr:192.168.65.3  Bcast:192.168.65.255  Mask:255.255.255.0
          inet6 addr: fe80::47ec:aa4f:d76:58a4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:210561 errors:0 dropped:0 overruns:0 frame:0
          TX packets:70488 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:305986209 (291.8 MiB)  TX bytes:3981660 (3.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:11382 errors:0 dropped:0 overruns:0 frame:0
          TX packets:11382 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:1035896 (1011.6 KiB)  TX bytes:1035896 (1011.6 KiB)

veth83bc651 Link encap:Ethernet  HWaddr A6:67:08:58:2D:FD  
          inet6 addr: fe80::a467:8ff:fe58:2dfd/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:19 errors:0 dropped:0 overruns:0 frame:0
          TX packets:27 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1806 (1.7 KiB)  TX bytes:2038 (1.9 KiB)

veth95cb2ed Link encap:Ethernet  HWaddr E6:19:AA:C1:9A:60  
          inet6 addr: fe80::e419:aaff:fec1:9a60/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:33 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:614 (614.0 B)  TX bytes:2388 (2.3 KiB)

veth97e341a Link encap:Ethernet  HWaddr B6:C9:3B:44:90:54  
          inet6 addr: fe80::b4c9:3bff:fe44:9054/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5 errors:0 dropped:0 overruns:0 frame:0
          TX packets:22 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:343 (343.0 B)  TX bytes:1601 (1.5 KiB)

```

#### Host Network

* Minimum network security level.
* No isolation on this type of open containers, thus leave tje container widely unprotected.
* Containers running in the host network stack should see a higher level of performance than those traversing the docker0 
bridge and iptables port mappings.

#### Overlay Network

* Supports multi-host networking out-of-the-box.
* Require some pre-existing conditions before it can be created.
    * Running Docker engine in Swarm mode.
    * A key-value store sucha as consul.

https://docs.docker.com/network/overlay-standalone.swarm/

### 31.- Define Container Networks with Doker Compose

```
git checkout -b v0.4
docker-compose up -d
docker network ls
docker-compose down
## Include my_net in the docker_compose
docker-compose up -d