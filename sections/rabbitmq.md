## RabbitMQ debug resources

### AMQP frame structure

```
                                     A single byte to
                                     determine the end
                                     of the frame
+------+-----------+------+---------+-----------------+
| Type | Channel # | Size | Payload | End-byte marker |
+------+-----------+------+---------+-----------------+
\-----------\/-----------/
       Frame header
```

#### Frame type
- Protocol header: Frame sent to establish a new connection between the 
broker (Rabbitmq) and a client.
- Method frame: Carries a RPC request or response.
- Content header: Certain specific methods carry a content `Basic.Publish`,
and the content header frame is used to send the properties of this content.
- Body: The frame with the actual content of your message
- Heartbeat: Used to confirm that a given client is still alive.

#### The Basic class

- Basic.Publish: sending messages from client to server, which happens asynchronously.
- Basic.Consume: starting consumers.
- Basic.Cancel: stopping consumers.
- Basic.Deliver & Basic.Return: sending messages from server to client, which happens asynchronously.
- Basic.Ack & Basic.Reject: Acknowledging messages.
- Basic.Get: Taking messages off the message queue synchronously.

#### Publishing and consuming messages

Client publishing a message:

```
+--------+                              +----------+
| Client |                              | RabbitMQ |
+--------+                              +----------+
    |                                        |
    |              Method frame              |
    |              Basic.Publish             |
    |--------------------------------------->|
    |                                        |
    |                                        |
    |                                        |
    |          Content Header frame          |
    |          content-type: plain/text      |
    |--------------------------------------->|
    |                                        |
    |                                        |
    |                                        |
    |                Body frame              |
    |            "The message body"          |
    |--------------------------------------->|
    |                                        |
```

Client consuming a messages:

```
+--------+                              +----------+
| Client |                              | RabbitMQ |
+--------+                              +----------+
    |                                        |
    |              Method frame              |
    |              Basic.Deliver             |
    |<---------------------------------------|
    |                                        |
    |                                        |
    |                                        |
    |          Content Header frame          |
    |          content-type: plain/text      |
    |<---------------------------------------|
    |                                        |
    |                                        |
    |                                        |
    |                Body frame              |
    |            "The message body"          |
    |<---------------------------------------|
    |                                        |
```

### Wireshark

#### Filters

Find AMQP (`amqp`) packets sent from a specific client (`tcp.srcport == 39488`)
the November 27th 2019 between 22:17:35 and 22:17:37 (`(frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")`):

```
amqp && tcp.srcport == 39488 && (frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")
```

#### Links

- [Inspecting AMQP 0-9-1 Traffic using Wireshark](https://www.rabbitmq.com/amqp-wireshark.html)
- [Wirshark AMQP filter reference](https://www.wireshark.org/docs/dfref/a/amqp.html)
- [AMQP and wireshark](https://wiki.wireshark.org/AMQP)

### TCPDump

```sh
$ IP_ADDR=$(netstat -tupln | grep \:5672) # to find out IP address
$ INTERFACE=$(ip ro | grep IP_ADDR) # to find out interface
$ tcpdump -i ${INTERFACE} port 5672 -w $(hostname)_rabbit.pcap # start the capture
```

### Tips

#### Dropping AMQP traffic

Drop all the AMQP traffic by drop packets with firewall (`iptables`):

```
$ sudo iptables -I 1 INPUT -p tcp --sport 5672 --dport 5672 --j DROP
```

Allow AMQP traffic:

```
$ sudo iptables -D INPUT 1
```

### Links

- [AMQP 0.9.1 specifications](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf)
- [A look into AMQP's frame structure](https://www.brianstorti.com/speaking-rabbit-amqps-frame-structure/)
