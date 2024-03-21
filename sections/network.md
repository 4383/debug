## Linux Networking Debug Resources

### Trace all network connections off a given process

```
$ strace -p <pid> -f -e trace=network -s 10000
```

### Trace all connections made on a specific network interface

... And store the records into a pcap file
```
$ sudo tcpdump -w tcpdump.pcap -i lo -tttt
```
