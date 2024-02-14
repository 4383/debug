## Eventlet debug resources

### Use tox as virtual env

You can run Eventlet's all its unit tests by using tox:

```
$ tox
```

The previous command will run unit tests with all the Eventlet hubs.

You can select a specific Eventlet hub (described in sections below) by using:

```
$ tox -e py312-asyncio
```

The previous command create `.tox/py312-asyncio/bin` directory that you
can access manually to trigger things in a more fine grained way.

### Run a Specific Unit Test Case

The following commands requires setup a tox environment first. See the
previous section.

Run all the tests of the wsgi test module:

```shell
$ .tox/py312-asyncio/bin/py.test tests/wsgi_test.py
```

Run a specific [unit test case](https://docs.python.org/3/library/unittest.html#unittest.TestCase) of the wsgi test module:

```shell
$ .tox/py312-asyncio/bin/py.test tests/wsgi_test.py::TestHttpd
```

Run an individual test belonging to a specific [unit test case](https://docs.python.org/3/library/unittest.html#unittest.TestCase) of the wsgi test module:

```shell
$ .tox/py312-asyncio/bin/py.test \
tests/wsgi_test.py::TestHttpd::test_close_idle_connections_listen_socket_closed
```

### Switch between hubs

Eventlet provide [various hubs](https://eventlet.readthedocs.io/en/latest/hubs.html). Hubs are designed to dispatches I/O events and schedules greenthreads.

Eventlet provide multiple hub implementations. The goal here is to see how to switch to one hub to another during your debug session.

Using Eventlet's [asyncio hub](https://eventlet.readthedocs.io/en/latest/migration.html#step-1-switch-to-the-asyncio-hub):

```shell
$ export EVENTLET_HUB=asyncio; \
.tox/py312-asyncio/bin/py.test tests/wsgi_test.py::TestHttpd
```

Using Eventlet's epoll hub:

```shell
$ export EVENTLET_HUB=epoll; \
.tox/py312-asyncio/bin/py.test tests/wsgi_test.py::TestHttpd
```

### Strace Unit Tests

[strace](https://strace.io/) is a diagnostic, debugging and
instructional userspace utility for Linux. It is used to monitor
and tamper with interactions between processes and the Linux kernel,
which include system calls, signal deliveries, and changes of process
state.

During your debug session you may want to `strace` your unit tests or a specific eventlet runtime. This section show you some examples of `strace` usages in an Eventlet context.

Strace a specific unit test from the [wsgi test module](https://github.com/eventlet/eventlet/blob/master/tests/wsgi_test.py):

```shell
$ export EVENTLET_HUB=asyncio
$ .tox/py312-asyncio/bin/py.test \
tests/wsgi_test.py::TestHttpd::test_close_idle_connections_listen_socket_closed  & \
strace -p $!
```

The `$!` parameter allow you to retrieve the process ID (pid) of the most recently executed process. As we put the execution of your unit test in background, the strace command should normally retrieve the pid of your unit test.

Running the previous command generated something like the following output:

```
accept4(15, 0x7ffe3e4855b0, [16], SOCK_CLOEXEC) = -1 EAGAIN (Resource temporarily unavailable)
fstat(15, {st_mode=S_IFSOCK|0777, st_size=0, ...}) = 0
epoll_ctl(11, EPOLL_CTL_ADD, 15, {EPOLLIN, {u32=15, u64=15}}) = 0
epoll_ctl(11, EPOLL_CTL_DEL, 14, 0x7ffe3e4869bc) = 0
getsockopt(14, SOL_SOCKET, SO_ERROR, [0], [4]) = 0
connect(14, {sa_family=AF_INET, sin_port=htons(45997), sin_addr=inet_addr("127.0.0.1")}, 16) = 0
epoll_wait(11, [], 2, 0)                = 0
mmap(NULL, 16384, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f3652da7000
setsockopt(16, SOL_TCP, TCP_QUICKACK, [1], 4) = 0
recvfrom(16, 0x557361928320, 8192, 0, NULL, NULL) = -1 EAGAIN (Resource temporarily unavailable)
fstat(16, {st_mode=S_IFSOCK|0777, st_size=0, ...}) = 0
epoll_ctl(11, EPOLL_CTL_ADD, 16, {EPOLLIN, {u32=16, u64=16}}) = 0
sendto(14, "PUT /foo-bar HTTP/1.1\r\nHost: loc"..., 62, 0, NULL, 0) = 62
epoll_wait(11, [{EPOLLIN, {u32=16, u64=16}}], 3, 0) = 1
sendto(14, "ABCABCABCABCABCABCABCABCABCABC", 30, 0, NULL, 0) = 30
epoll_ctl(11, EPOLL_CTL_DEL, 16, 0x7ffe3e48504c) = 0
recvfrom(16, "PUT /foo-bar HTTP/1.1\r\nHost: loc"..., 8192, 0, NULL, NULL) = 92
getsockname(16, {sa_family=AF_INET, sin_port=htons(45997), sin_addr=inet_addr("127.0.0.1")}, [16]) = 0
sendto(16, "HTTP/1.1 200 OK\r\nContent-Type: a"..., 164, 0, NULL, 0) = 164
recvfrom(16, 0x557361928320, 8192, 0, NULL, NULL) = -1 EAGAIN (Resource temporarily unavailable)
fstat(16, {st_mode=S_IFSOCK|0777, st_size=0, ...}) = 0
epoll_ctl(11, EPOLL_CTL_ADD, 16, {EPOLLIN, {u32=16, u64=16}}) = 0
epoll_wait(11, [], 3, 0)                = 0
recvfrom(14, "HTTP/1.1 200 OK\r\nContent-Type: a"..., 8192, 0, NULL, NULL) = 164
epoll_wait(11, [], 3, 0)                = 0
shutdown(15, SHUT_RDWR)                 = 0
epoll_wait(11, [{EPOLLHUP, {u32=15, u64=15}}], 3, 0) = 1
close(15)                               = 0
epoll_ctl(11, EPOLL_CTL_DEL, 15, 0x7ffe3e48564c) = -1 EBADF (Bad file descriptor)
accept4(-1, 0x7ffe3e4855b0, [16], SOCK_CLOEXEC) = -1 EBADF (Bad file descriptor)
accept4(-1, 0x7ffe3e4855b0, [16], SOCK_CLOEXEC) = -1 EBADF (Bad file descriptor)
...
accept4(-1, 0x7ffe3e4855b0, [16], SOCK_CLOEXEC) = -1 EBADF (Bad file descriptor)
```

We can observe that something went wrong during the execution of this test
when using the Eventlet asyncio hub.

To avoid unexpected pid creation I'd suggest to use a container to run your unit tests and to isolate your process in an environment with a low system activity level. You can use my [machine project](https://github.com/4383/machine) to setup this isolated environment. Here below is a demo of this kind of environment.

[![asciicast](https://asciinema.org/a/0e7cLRsHiNJlBQdlIgdkApAsU.svg)](https://asciinema.org/a/0e7cLRsHiNJlBQdlIgdkApAsU)

### Profile Unit Tests

Profiling a specific unit test (`test_001_server`) from the wsgi test module:

```
$ export EVENTLET_HUB=asyncio; \
.tox/py312-asyncio/bin/python -m cProfile -o profiling.cprof \
.tox/py312-asyncio/bin/py.test \
tests/wsgi_test.py::TestHttpd::test_001_server
```

You can even profile and `strace` this test at the same time:

```
$ export EVENTLET_HUB=asyncio; \
.tox/py312-asyncio/bin/python -m cProfile -o profiling.cprof \
.tox/py312-asyncio/bin/py.test \
tests/wsgi_test.py::TestHttpd::test_001_server & \
strace -p $!
```

Once done, you can use [pyprof2calltree](https://pypi.org/project/pyprof2calltree/)
command to convert your profiling data into a format that can be opened with
[KCacheGrind](https://kcachegrind.github.io/html/Home.html). Example:

```
$ export EVENTLET_HUB=asyncio; \
.tox/py312-asyncio/bin/python -m cProfile -o profiling.cprof \
.tox/py312-asyncio/bin/py.test \
tests/wsgi_test.py::TestHttpd::test_001_server & \
strace -p $!
$ pyprof2calltree -k -i profiling.cprof
```

The previous last command opened KCacheGrind with our data:

![KCacheGrind of eventlet unit test](kcachegrind-eventlet.png "KCacheGrind of eventlet unit test")


### Links

- [Eventlet Documentation](https://eventlet.readthedocs.io/)
