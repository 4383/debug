## Linux Networking Debug Resources

### Trace all network connections off a given process

```sh
$ strace -p <pid> -f -e trace=network -s 10000
```

### Trace all connections made on a specific network interface

... And store the records into a pcap file

```sh
$ sudo tcpdump -w tcpdump.pcap -i lo -tttt
```

### Show network connections made by a Nogi audit

Lets consider that the process id (`pid`) of our running Nogi based app is
`1871958`.

```sh
$ sudo netstat -4 -6 -a -n -p | grep 1871958
tcp        0      0 0.0.0.0:5556            0.0.0.0:*               LISTEN      1871958/python
```

List file descriptors opened by this process:

```sh
$ ls -la /proc/1871958/fd
total 0
dr-x------. 2 hberaud hberaud 12 Mar 22 09:42 .
dr-xr-xr-x. 9 hberaud hberaud  0 Mar 22 09:41 ..
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 0 -> /dev/pts/12
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 1 -> /dev/pts/12
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 10 -> 'socket:[13098209]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 11 -> 'socket:[13097317]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 12 -> 'socket:[13103797]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 2 -> /dev/pts/12
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 4 -> 'anon_inode:[eventfd]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 5 -> 'anon_inode:[eventfd]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 6 -> 'anon_inode:[eventpoll]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 7 -> 'anon_inode:[eventfd]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 8 -> 'anon_inode:[eventpoll]'
lrwx------. 1 hberaud hberaud 64 Mar 22 09:42 9 -> 'anon_inode:[eventfd]'
```

Now lets list all files opened by this process:

```sh
$ lsof -a -p 1871958
COMMAND     PID    USER   FD      TYPE   DEVICE SIZE/OFF   NODE NAME
python  1871958 hberaud  cwd       DIR     0,37      264 544378 /home/hberaud/dev/nogi
python  1871958 hberaud  rtd       DIR     0,33      158    256 /
python  1871958 hberaud  txt       REG     0,33    15880 218749 /usr/bin/python3.12
python  1871958 hberaud  mem       REG     0,31          218749 /usr/bin/python3.12 (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          204398 /usr/lib64/libnss_resolve.so.2 (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          204296 /usr/lib64/libstdc++.so.6.0.32 (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          204377 /usr/lib64/libcap.so.2.48 (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          204397 /usr/lib64/libnss_myhostname.so.2 (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          551006 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/utils.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551020 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/error.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551011 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_version.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551017 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_poll.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          217786 /usr/lib64/python3.12/lib-dynload/_pickle.cpython-312-x86_64-linux-gnu.so (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          551012 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/message.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551013 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/socket.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551015 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/context.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          550884 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/pyzmq.libs/libsodium-cb25555f.so.23.3.0 (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551004 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_proxy_steerable.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
...
python  1871958 hberaud  mem       REG     0,31          303544 /usr/lib64/gconv/gconv-modules.cache (path dev=0,33)
python  1871958 hberaud  mem       REG     0,31          203825 /usr/lib64/ld-linux-x86-64.so.2 (path dev=0,33)
python  1871958 hberaud    0u      CHR   136,12      0t0     15 /dev/pts/12
python  1871958 hberaud    1u      CHR   136,12      0t0     15 /dev/pts/12
python  1871958 hberaud    2u      CHR   136,12      0t0     15 /dev/pts/12
python  1871958 hberaud    3u     IPv4 13104010      0t0    TCP fedora:45552->text-lb.drmrs.wikimedia.org:https (ESTABLISHED)
python  1871958 hberaud    4u  a_inode     0,15        0   1050 [eventfd:290]
python  1871958 hberaud    5u  a_inode     0,15        0   1050 [eventfd:501]
python  1871958 hberaud    6u  a_inode     0,15        0   1050 [eventpoll:5]
python  1871958 hberaud    7u  a_inode     0,15        0   1050 [eventfd:502]
python  1871958 hberaud    8u  a_inode     0,15        0   1050 [eventpoll:7,10,11]
python  1871958 hberaud    9u  a_inode     0,15        0   1050 [eventfd:534]
python  1871958 hberaud   10u     IPv4 13098209      0t0    TCP *:freeciv (LISTEN)
python  1871958 hberaud   11u     IPv4 13097317      0t0    TCP localhost:freeciv->localhost:58122 (ESTABLISHED)
python  1871958 hberaud   12u     IPv4 13104000      0t0    TCP fedora:45548->text-lb.drmrs.wikimedia.org:https (ESTABLISHED)
```

In 3 previous output We can observe that your zmq publish socket is listed.
It correspond to the `10` file descriptor. Its state is `LISTEN`.

An other interesting things that we can observe within the last command
output are the listed shared libraries (all the `.so` files). We can
use inspect them to retrieve their symbols and so use them in conjunction
with eBPF and `bpftrace`.

Example of symbols inspection:

```sh
$ lsof -a -p 1871958 | grep zmq
python  1871958 hberaud  mem       REG     0,31          551006 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/utils.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551020 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/error.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551011 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_version.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551017 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_poll.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551012 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/message.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551013 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/socket.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551015 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/context.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          550884 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/pyzmq.libs/libsodium-cb25555f.so.23.3.0 (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551004 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_proxy_steerable.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          550883 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/pyzmq.libs/libzmq-f468291a.so.5.2.4 (path dev=0,37)
python  1871958 hberaud  mem       REG     0,31          551005 /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/zmq/backend/cython/_device.cpython-312-x86_64-linux-gnu.so (path dev=0,37)
```

There are listed all the zmq related shared libraries.

Now lets inspect which symbols of the libzmq are associated with packet sending:

```sh
$ nm -D /home/hberaud/dev/nogi/.venv/lib/python3.12/site-packages/pyzmq.libs/libzmq-f468291a.so.5.2.4 | egrep 'send|size'
                 U send@GLIBC_2.2.5
                 U sendto@GLIBC_2.2.5
0000000000098793 T zmq_msg_init_size
0000000000098822 T zmq_msg_send
0000000000098934 T zmq_msg_size
0000000000097baa T zmq_send
0000000000097cdc T zmq_send_const
0000000000097e0e T zmq_sendiov
0000000000097b7f T zmq_sendmsg
                 U _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE4sizeEv@GLIBCXX_3.4.21
```

And then we are now able to trace all the calls made to this shared library:

```sh
ltrace -l libzmq-f468291a.so.5.2.4 -p 1871958
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 57
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 78
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 115
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 112
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 78
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)                                                           = 284
libzmq-f468291a.so.5.2.4->zmq_msg_size(0x7f03701ff4a0, 0x7f03701ff4a0, 0, 0x7f03701ff4a0)
```

All these commands are really useful to observe what happens in a running app.
