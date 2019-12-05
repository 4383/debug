## RabbitMQ debug resources

### Wireshark

Find AMQP (`amqp`) packets sent from a specific client (`tcp.srcport == 39488`)
the November 27th 2019 between 22:17:35 and 22:17:37 (`(frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")`):

```
amqp && tcp.srcport == 39488 && (frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")
```

### TCPDump

```sh
$ IP_ADDR=$(netstat -tupln | grep \:5672) # to find out IP address
$ INTERFACE=$(ip ro | grep IP_ADDR) # to find out interface
$ tcpdump -i ${INTERFACE} port 5672 -w $(hostname)_rabbit.pcap # start the capture
```
