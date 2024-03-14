## Python Debug Resources

In the following examples I refer to one of my code snippets
named [atomic.py](https://github.com/4383/snippets/blob/main/python/atomic.py).

I also use my [preconfigured environment](https://github.com/4383/machine),
with all the apps and commands used already installed.

Do not hesistate to checkout my boths repos to test the following commands.

### Using GDB with python

Using GDB can avoid to edit your code to append debug points.

In the following command, I first ran my script, and then I directly attach
gdb to the pid corresponding to my running python script:

```
python python/atomic.py & gdb python $!
```

### Retrieve the Current Exception in PDB

`pdb` stores the exception type and value in `__exception__`.

You can simply retrieve this exception by using:

```python
(pdb) __exception__
```

You can print the exception part of a traceback in pdb with:

```python
(Pdb) import traceback; print "".join(traceback.format_exception_only(*__exception__))
```

This trick is useful when you deal with code like the following snippet:

```python
try:
    do_something()
except:
    do_something_else()
```

### Compile Python Code

Sometime it could be useful to compile python.
To do that you can simply use the [pyinstaller library](https://pyinstaller.org/en/stable/).

Compiling the code of `atomic.py`:

```
$ pyinstaller atomic.py
```

The previous command will generate a `dist` folder containing something like the following (truncated) output:

```
dist
└── atomic
    ├── _internal
    │   ├── base_library.zip
    │   ├── lib-dynload
    │   │   ├── _bisect.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _blake2.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _bz2.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _codecs_cn.cpython-312-x86_64-linux-gnu.so
...
    │   │   ├── _datetime.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _decimal.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _hashlib.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _heapq.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _lzma.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _md5.cpython-312-x86_64-linux-gnu.so
    │   │   ├── _multibytecodec.cpython-312-x86_64-linux-gnu.so
...
    │   │   ├── _random.cpython-312-x86_64-linux-gnu.so
    │   │   └── zlib.cpython-312-x86_64-linux-gnu.so
    │   ├── libbz2.so.1.0
    │   ├── libcrypto.so.1.1
    │   ├── liblzma.so.5
    │   ├── libpython3.12.so.1.0
    │   └── libz.so.1
    └── atomic

3 directories, 44 files
```

### Disassemble Compiled Python Code

Compiling and disassembling python code can allow you to implement
some sort of linters, like one that could be used to identify
file handeling made at the `C` level with [`fopen`](https://www.tutorialspoint.com/c_standard_library/c_function_fopen.htm).

Image you want to migrate existing blocking code into async code,
maybe by using `asyncio`, and you want to identify first which part of
your code could be source of blocking IO, compiling and then decompiling
Python code can help you identifying where are located the code you
need to reformat.

Your code may use underlying libraries where are hosted your blocking
IOs, by compiling and decompiling you transform your code into low level
code with a direct access to your hidden blocking IOs.

Based on the previously generated binary (see the previous section),
the following command will dissasemble this binary:

```
$ objdump -d dist/atomic/atomic
```

```
dist/atomic/atomic:     file format elf64-x86-64


Disassembly of section .init:

0000000000402000 <.init>:
  402000:	48 83 ec 08          	sub    $0x8,%rsp
  402004:	48 8b 05 ed bf 00 00 	mov    0xbfed(%rip),%rax        # 40dff8 <dlerror@plt+0xbad8>
  40200b:	48 85 c0             	test   %rax,%rax
  40200e:	74 05                	je     402015 <__strcat_chk@plt-0x1b>
  402010:	e8 ab 02 00 00       	callq  4022c0 <__gmon_start__@plt>
  402015:	48 83 c4 08          	add    $0x8,%rsp
  402019:	c3                   	retq   

Disassembly of section .plt:

0000000000402020 <__strcat_chk@plt-0x10>:
  402020:	ff 35 e2 bf 00 00    	pushq  0xbfe2(%rip)        # 40e008 <dlerror@plt+0xbae8>
  402026:	ff 25 e4 bf 00 00    	jmpq   *0xbfe4(%rip)        # 40e010 <dlerror@plt+0xbaf0>
  40202c:	0f 1f 40 00          	nopl   0x0(%rax)

0000000000402030 <__strcat_chk@plt>:
  402030:	ff 25 e2 bf 00 00    	jmpq   *0xbfe2(%rip)        # 40e018 <dlerror@plt+0xbaf8>
  402036:	68 00 00 00 00       	pushq  $0x0
  40203b:	e9 e0 ff ff ff       	jmpq   402020 <__strcat_chk@plt-0x10>

0000000000402040 <getenv@plt>:
  402040:	ff 25 da bf 00 00    	jmpq   *0xbfda(%rip)        # 40e020 <dlerror@plt+0xbb00>
  402046:	68 01 00 00 00       	pushq  $0x1
  40204b:	e9 d0 ff ff ff       	jmpq   402020 <__strcat_chk@plt-0x10>
...

0000000000402460 <fopen@plt>:
  402460:	ff 25 ca bd 00 00    	jmpq   *0xbdca(%rip)        # 40e230 <dlerror@plt+0xbd10>
  402466:	68 43 00 00 00       	pushq  $0x43
  40246b:	e9 b0 fb ff ff       	jmpq   402020 <__strcat_chk@plt-0x10>

...
  407e14:	e8 47 a6 ff ff       	callq  402460 <fopen@plt>
  407e19:	48 81 c4 08 10 00 00 	add    $0x1008,%rsp
  407e20:	5d                   	pop    %rbp
  407e21:	41 5c                	pop    %r12
  407e23:	c3                   	retq   
  407e24:	0f 1f 40 00          	nopl   0x0(%rax)
  407e28:	8b 05 6e 64 00 00    	mov    0x646e(%rip),%eax        # 40e29c <dlerror@plt+0xbd7c>
  407e2e:	83 f8 ff             	cmp    $0xffffffff,%eax
  407e31:	74 3d                	je     407e70 <dlerror@plt+0x5950>
  407e33:	85 c0                	test   %eax,%eax
  407e35:	0f 85 84 00 00 00    	jne    407ebf <dlerror@plt+0x599f>
  4086ee:	e8 0d 99 ff ff       	callq  402000 <__strcat_chk@plt-0x30>
  4086f3:	48 85 ed             	test   %rbp,%rbp
  4086f6:	74 1e                	je     408716 <dlerror@plt+0x61f6>
  4086f8:	0f 1f 84 00 00 00 00 	nopl   0x0(%rax,%rax,1)
  4086ff:	00 
  408700:	4c 89 ea             	mov    %r13,%rdx
  408703:	4c 89 f6             	mov    %r14,%rsi
  408706:	44 89 ff             	mov    %r15d,%edi
  408709:	41 ff 14 dc          	callq  *(%r12,%rbx,8)
  40870d:	48 83 c3 01          	add    $0x1,%rbx
  408711:	48 39 eb             	cmp    %rbp,%rbx
  408714:	75 ea                	jne    408700 <dlerror@plt+0x61e0>
  408716:	48 83 c4 08          	add    $0x8,%rsp
  40871a:	5b                   	pop    %rbx
  40871b:	5d                   	pop    %rbp
  40871c:	41 5c                	pop    %r12
  40871e:	41 5d                	pop    %r13
  408720:	41 5e                	pop    %r14
  408722:	41 5f                	pop    %r15
  408724:	c3                   	retq   
  408725:	90                   	nop
  408726:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
  40872d:	00 00 00 
  408730:	f3 c3                	repz retq 

Disassembly of section .fini:

0000000000408734 <.fini>:
  408734:	48 83 ec 08          	sub    $0x8,%rsp
  408738:	48 83 c4 08          	add    $0x8,%rsp
  40873c:	c3                   	retq   
```

### USDT (User Statically-Defined Tracing (USDT) probes) & Python

Imagine you ran a python server, and you want to monitor threads for this
running python process.

First, lets start this server:

```
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```


You can use [bpftrace](https://github.com/bpftrace/bpftrace) to list all USDT
available for this running python server:

```
$ sudo bpftrace -lp <pid> "usdt:*" 
...
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:mutex_timedlock_acquired
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:mutex_timedlock_entry
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:pthread_create
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:pthread_join
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:pthread_join_ret
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:pthread_start
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:rdlock_acquire_read
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:rdlock_entry
usdt:/proc/920960/root/usr/lib64/libc.so.6:libc:rwlock_destroy
...
```

Replace `<pid>` the pid of the running python server that you want to inspect,
you can retrieve this `pid` by using `ps ax | grep python`.

The previous command list all the USDT you can attach.

We want to observe threads, so lets focus on them:

```
$ sudo bpftrace -p <pid> -e \
    'usdt:/proc/<pid>/root/usr/lib64/libc.so.6:libc:pthread* 
    { printf("%s %u [%u] %u %s\n", comm, pid, cpu, elapsed, probe); }'
Attaching 4 probes...
python3 <pid> [3] 1546728119 usdt:/proc/921304/root/usr/lib64/libc.so.6:libc:pthread_create
python3 <pid> [1] 1546799291 usdt:/proc/921304/root/usr/lib64/libc.so.6:libc:pthread_start
```

In the example above we can observe that a thread is created and started when
I submit a new request to this running server (`wget http://0.0.0.0:8000/`).

Lot of USDT endpoints can be monitored, that can be useful to debug your apps.
