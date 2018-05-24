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