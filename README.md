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

```
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
```

docker run -d --name container_2 busybox sleep 1000
docker exec -it container_2 ifconfig

```
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
```

```