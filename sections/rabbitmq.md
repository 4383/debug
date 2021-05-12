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

### RabbitMQ Ports

This section define classic ports in use in a traditional way and in
a clustering way (HA).

For a cluster of nodes, they must be open to each other on 35197,
4369 and 5672.

See [networking guide](https://www.rabbitmq.com/networking.html#ports)
for more details.

#### Port 4369

RabbitMQ use the port 4369 for discovery (
[Erlang Port Mapper Daemon (epmd)](http://erlang.org/doc/man/epmd.html)).
Discovery mean resolution of node names in a cluster. Nodes must be able
to reach each other and the port mapper daemon for clustering to work.

#### Port 35197

The port 35197 is used to run distribued Erlang through a firewall.

Don't forget that RabbitMQ is wrote in Erlang.

Port 35197 set by `inet_dist_listen_min/max` firewalls must permit traffic
in this range to pass between clustered nodes.

#### Port 5672

RabbitMQ nodes and server talk to clients via the port 5672 (default port).

For any servers that want to use the message queue, only 5672 is required.

#### Port 25672

RabbitMQ nodes talk to each other over the port 25672 (clustering port)

Used for inter-node and CLI tools communication
(Erlang distribution server port) and is allocated from a dynamic range
(limited to a single port by default, computed as AMQP port + 20000).
Unless external connections on these ports are really necessary
(e.g. the cluster uses federation or CLI tools are used on machines
outside the subnet), these ports should not be publicly exposed.

#### Port 15672 & 55672

These ports (15672 and 55672) are used by the management console.

- Port 15672 for RabbitMQ version 3.x
- Port 55672 for RabbitMQ pre 3.x

#### Which ports to debug

Depends on your context, if you try to debug a client server communcation
like `oslo.messaging` and RabbitMQ cluter in openstack, then you only need
to capture the port `5672`. See the `tcpdump` section below for detailed
example.

If you try to debug some synchronization or
[mirroring](https://www.rabbitmq.com/ha.html) between your cluster nodes then
it can be useful to capture the traffic on ports `25672`, `4369`, and `35197`.
This traffic analyze can help you to understand if which packets transit
between your nodes and why your error happen.

See the [clustering guide](https://www.rabbitmq.com/clustering.html)
for more informations.

### Wireshark

All the below resources are related to AMQP version 0.9

#### Methods matrix

List of binding used by the wireshark dissector to represent AMQP Basic methods:

```
+---------------------+-------+
| Method              | Value |
+---------------------+-------+
| BASIC.QOS           |  10   |
| BASIC.QOS_OK        |  11   |
| BASIC.CONSUME       |  20   |
| BASIC.CONSUME_OK    |  21   |
| BASIC.CANCEL        |  30   |
| BASIC.CANCEL_OK     |  31   |
| BASIC.PUBLISH       |  40   |
| BASIC.RETURN        |  50   |
| BASIC.DELIVER       |  60   |
| BASIC.GET           |  70   |
| BASIC.GET_OK        |  71   |
| BASIC.GET_EMPTY     |  72   |
| BASIC.ACK           |  80   |
| BASIC.REJECT        |  90   |
| BASIC.RECOVER_ASYNC | 100   |
| BASIC.RECOVER       | 110   |
| BASIC.RECOVER_OK    | 111   |
| BASIC.NACK          | 120   |
+---------------------+-------+
```

You can refer these values in filters by using the filter `amqp.method.method`.

Example by filtering on AMQP `Basic.Publish` packets:

```
amqp.method.method == 40
```

Example by filtering on AMQP `Basic.Consume` packets:

```
amqp.method.method == 20
```

See the previous client and server schema and see the section below for more
filter examples.

#### Filters

Find all packets who correspond to an AMQP client which publish a message on broker:

```
amqp.method.class == 60 && amqp.method.method == 40
```

Find AMQP (`amqp`) packets sent from a specific client (`tcp.srcport == 39488`)
the November 27th 2019 between 22:17:35 and 22:17:37 (`(frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")`):

```
amqp && tcp.srcport == 39488 && (frame.time >=  "Nov 27, 2019 22:17:35") && (frame.time <=  "Nov 27, 2019 22:17:37")
```

#### Links

- [Inspecting AMQP 0-9-1 Traffic using Wireshark](https://www.rabbitmq.com/amqp-wireshark.html)
- [Wirshark AMQP filter reference](https://www.wireshark.org/docs/dfref/a/amqp.html)
- [AMQP and wireshark](https://wiki.wireshark.org/AMQP)
- [Wireshark dissector source code](https://github.com/wireshark/wireshark/blob/master/epan/dissectors/packet-amqp.c)
- [Wireshark dissector defines](https://github.com/wireshark/wireshark/blob/master/epan/dissectors/packet-amqp.c#L98,L418)

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

### Activate debug mode on openstack/oslo

```
[DEFAULT]
debug=true
default_log_levels=kombu=DEBUG,oslo.messaging=DEBUG,py-amqp=DEBUG"
```

### Links

- [AMQP 0.9.1 specifications](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf)
- [A look into AMQP's frame structure](https://www.brianstorti.com/speaking-rabbit-amqps-frame-structure/)
