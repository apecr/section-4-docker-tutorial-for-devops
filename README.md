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