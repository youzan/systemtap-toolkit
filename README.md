[systemtap]: https://sourceware.org/systemtap/
[agentzh]: https://github.com/openresty/nginx-systemtap-toolkit#sample-bt
[@agentzh]: https://github.com/agentzh
[@brendangregg]: https://github.com/brendangregg
[@fche]: https://web.elastic.org/~fche/
[@youzan]: https://github.com/youzan
[dwarf]: http://www.dwarfstd.org/
[tcpguide]: http://www.tcpipguide.com/free/t_TCPOperationalOverviewandtheTCPFiniteStateMachineF-2.htm

<h1 align="center">systemtap-toolkit</h1>

<p align="center">
  <img src="http://image.slidesharecdn.com/tracingsummit2014fromdtracetolinux-141013073949-conversion-gate02/95/from-dtrace-to-linux-56-638.jpg"/>
  <br />
</p>


NAME
====
systemtap-toolkit

Description
===========

This is [@YouZan] systemtap toolkit to online analyze the complicated problem on production with heavy load. All tools are based on the amazing linux tracing/probing tool [systemtap].

Any guys which want to know  what the hell it is in the user space and kernel space should be to learn [systemtap] which is awesome tool:)

Table of Contents
=================

* [NAME](#name)
* [Description](#description)
* [Requirements](#requirements)
* [Contribute](#contribute)
* [Thanks](#thanks)
* [tcp-passive-syn-ack-time](#tcp-passive-syn-ack-time)
* [tcp-active-syn-ack-time](#tcp-active-syn-ack-time)
* [tcp-retrans](#tcp-retrans)
* [who-open-file](#who-open-file)
* [who-ctxswitch-process](#who-ctxswitch-process)
* [syscall-connect](#syscall-connect)
* [sample-bt](#sample-bt)
* [watch-var](#watch-var)
* [tcp-trace-packet](#tcp-trace-packet)
* [ngx-req-watch](#ngx-req-watch)
* [stracelike](#stracelike)
* [redis-watch-req](#redis-watch-req)
* [libcurl-watch-req](#libcurl-watch-req)
* [pdomysql-watch-query](#pdomysql-watch-query)
* [io-process-top](#io-process-top)
* [net-process-top](#net-process-top)
* [phpredis-watch-req](#phpredis-watch-req)
* [nssdns-watch-question](#nssdns-watch-question)
* [phpfpm-watch-req](#phpfpm-watch-req)
* [swoole-redis-watch](#swoole-redis-watch)


requirements
============

We need [systemtap] and [dwarf].
some scripts are working on kernel space and other is working on the user space.

For kernel space, we need kernel debuginfo like `kernel-debuginfo-3.10.0-327.28.3.el7.x86_64`.

For user space, we need user application debuginfo like `redis-debuginfo-2.8.19-2.el7.x86_64`.

For *redhat*\* linux version, we can install as the following:

```bash
yum install yum-utils #for debuginfo-install
yum install systemtap
yum install kernelname-devel-version
debuginfo-install kernelname-version
```


contribute
==========
You can choose the ways as the following to help this project.

1. To contribute to this project, clone this repo locally and commit your code on a separate branch.
2. Create Github issues.
3. You can reach me detailyang@gmail.com.

thanks
==========
Special thanks to [@brendangregg]、 [@agentzh] and [@fche]. All we have learn for [systemtap] is from their amazing blog posts and projects:)

tcp-passive-syn-ack-time
===============
It's used to measure the time of syn packet to ack packet on the server side in the tcp-3-shakehands(Thanks [tcpguide]).

````bash
[root@localhost tmp]# ./tcp-passive-syn-ack-timee -p 80 -t 5000
Collecting tcp dport (80)...syn-ack time

interval min:197us, max:858us avg:519us, cnt:3
value |-------------------------------------------------- count
   32 |                                                   0
   64 |                                                   0
  128 |@                                                  1
  256 |@                                                  1
  512 |@                                                  1
 1024 |                                                   0
 2048 |                                                   0
````



tcp-active-syn-ack-time
===============
It's used to measure the time of syn packet to ack packet on the client side in the tcp-3-shakehands(Thanks [tcpguide]).

````bash
[root@localhost systemtap-toolkit]# ./tcp-active-syn-ack-time -p 80 -t 5000
Collecting tcp dport (80)...syn-ack time

dport:80 min:417us, max:542us avg:460us, cnt:3
value |-------------------------------------------------- count
   64 |                                                   0
  128 |                                                   0
  256 |@@                                                 2
  512 |@                                                  1
 1024 |                                                   0
 2048 |                                                   0

````

tcp-retrans
===========
It's used to collecting which tcp packet being retransmit

````bash
[root@localhost systemtap-toolkit]# ./tcp-retrans
Printing tcp retransmission

10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:0 -> 1000 ms
10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:1000 -> 2000 ms
10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:2000 -> 4000 ms
````

who-open-file
=============
It's used to find who is opening the specified file

````bash
[root@localhost systemtap-toolkit]# ./who-open-file -f 123 -t 10000
Collecting who is opening filename 123

cat(13740) is opening the filename: "123"
cat(13741) is opening the filename: "123"
````

who-ctxswitch-process
=====================
Tracing context switch for specified process.

```bash
[root@localhost systemtap-toolkit]# ./who-ctxswitch-process -p 6354
Collecting who is context switch 6354
[0] swapper/0       (    0)<R>           => nginx           ( 6354)<R>
[0] nginx           ( 6354)<S>           => nginx           ( 6355)<R>
[0] nginx           ( 6355)<D>           => nginx           ( 6354)<R>
[0] nginx           ( 6354)<S>           => rcu_sched       (   10)<R>
[0] nginx           ( 6355)<D>           => nginx           ( 6354)<R>
```

syscall-connect
==============
It's used to tracing syscall.connect

````bash
telnet(8062) is connecting to AF_INET@192.168.33.10:1800
telnet(8063) is connecting to AF_INET@192.168.33.10:1800
telnet(8064) is connecting to AF_INET@192.168.33.10:1800
telnet(8065) is connecting to AF_INET@192.168.33.10:1800
telnet(8066) is connecting to AF_INET@192.168.33.10:1800
telnet(8067) is connecting to AF_INET@192.168.33.10:1800
telnet(8068) is connecting to AF_INET@192.168.33.10:1800
telnet(8069) is connecting to AF_INET@192.168.33.10:1800
telnet(8070) is connecting to AF_INET@192.168.33.10:1800
````

sample-bt
=========
It's from [agentzh] and be used to sampling the backtrace in the user space and kernel space.

````bash
$ ./sample-bt -p 8736 -t 5 -u > a.bt
WARNING: Tracing 8736 (/opt/nginx/sbin/nginx) in user-space only...
WARNING: Missing unwind data for module, rerun with 'stap -d stap_df60590ce8827444bfebaf5ea938b5a_11577'
WARNING: Time's up. Quitting now...(it may take a while)
WARNING: Number of errors: 0, skipped probes: 24
````

watch-var
=========
It's used to monitor function param changing.

````bash
[root@localhost systemtap-toolkit]# ./watch-var  -f syscall.open -v filename -p 25849
WARNING: Tracing vars syscall.open filename in 25849...
a.out[25849] kernel.function("SyS_open@fs/open.c:1036").call filename: "" => ""./test""
````

tcp-trace-packet
================
Like tcpdump, it's used to tracing tcp packet with more detail include tcp flag.

````bash
[root@localhost systemtap-toolkit]# ./tcp-trace-packet
WARNING: tracking 0 tcp packet
1478067249998698 10.0.2.15:22 => 10.0.2.2:50627 len:92 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067249998955 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
1478067250199252 10.0.2.15:22 => 10.0.2.2:50627 len:172 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067250199559 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
1478067250399756 10.0.2.15:22 => 10.0.2.2:50627 len:100 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067250399963 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
````

ngx-req-watch
===============
It tracing the userland, which can watch and filter by specified condition nginx request in real time

````bash
[root@localhost systemtap-toolkit]# ./ngx-req-watch -p 5614
WARNING: watching /opt/tengine/sbin/nginx(8521 8522 8523 8524) requests
nginx(8523) GET URI:/123?a=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?a=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?a=123&b=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w HOST:test STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w=a HOST:test STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
````

stracelike
==============
Like strace. But it's based on the [systemtap]
````bash
[root@localhost systemtap-toolkit]# ./stracelike -p 4580 -t 20000
WARNING: stracing syscall
Sat Oct 29 12:46:19 2016.094410  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.100334>
Sat Oct 29 12:46:19 2016.194756  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.100227>
Sat Oct 29 12:46:19 2016.295006  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.101086>
````

redis-watch-req
===============
It tracing the userland, which can watch and filter by specified condition redis request in real time

````bash
[root@localhost systemtap-toolkit]# ./redis-watch-req -p 23261
WARNING: watching /usr/bin/redis-server(23261) requests
redis-server(23261) RT:30(us) REQ: id:2 fd:5 ==> get a #-1 RES: #9
redis-server(23261) RT:23(us) REQ: id:2 fd:5 ==> set a #12 RES: #5
redis-server(23261) RT:16(us) REQ: id:2 fd:5 ==> get foo #-1 RES: #5
````

libcurl-watch-req
=================
It traceing the userland, which can watch and filter by specified condition request for softawre which are based on the libcurl like `curl` and `php`.

````bash
[root@localhost systemtap-toolkit]# ./libcurl-watch-req
WARNING: Tracing libcurl (0) ...
curl(23759) URL:http://www.google.com RT:448(ms) RTCODE:0
curl(23767) URL:http://www.facebook.com/asdfasdf RT:596(ms) RTCODE:0
curl(23769) URL:https://www.facebook.com/asdfasdf RT:902(ms) RTCODE:0
````

pdomysql-watch-query
=================
It traceing the userland, which can watch and filter by specified condition request for php's pdo mysql driver.

````bash
[root@localhost systemtap-toolkit]# ./pdomysql-watch-query -l /usr/lib64/php/modules/pdo_mysql.so

Tracing pdo-mysql (0)
php-fpm(12896) 172.17.10.196:3306@root: SELECT * from person RT:0(ms) RTCODE:1
php-fpm(12896) 172.17.10.196:3306@root: SELECT * from person RT:8(ms) RTCODE:1
php-fpm(12896)172.17.10.196:3306@root: SELECT sleep(5) RT:5012(ms) RTCODE:1
````

phpredis-watch-req
=================
It traceing the userland, which can trace the php redis request

```bash
[root@localhost systemtap-toolkit]# ./phpredis-watch-req -l /usr/lib64/php/modules/redis.so

Tracing phpredis (/usr/lib64/php/modules/redis.so)

php(17226)<zim_Redis___construct[22us]>
php(17226)<zim_Redis_connect[113us]>
php(17226)<zim_Redis_get[157us]>:*2 $3 GET $3 key
php(17226)<zim_Redis_hGet[563us]>:*3 $4 HGET $3 key $6 ffffff
php(17226)<zim_Redis_set[617us]>:*3 $3 SET $3 key $4 abcd
php(17226)<zim_Redis___destruct[12us]>
```

io-process-top
=================
It traceing io Read|Write with the view of process(pid).

````bash
[root@localhost systemtap-toolkit]# ./io-process-top  -t 1000
WARNING: Collecting IO Process Top 10 with interval of 1000ms
        Process Name          Read(KB)   Write(KB)
        redis-server(4510)            3           0
              stapio(28280)           2           0
     systemd-journal(442)             0           0
             systemd(1)               0           0
                sshd(19948)           0           0
        in:imjournal(595)             0           0
```

net-process-top
=================
It traceing net Send|Recv with the view of process(pid).

```bash
[root@localhost systemtap-toolkit]# ./net-process-top -t 5000
WARNING: Collecting Net Process Top 10 with interval of 5000ms
             Process(    0)    dev   Send(PK)   Recv(PK)   Send(KB)   Recv(KB)
               nginx( 7266)     lo     446203          0     144471          0
                 wrk(27496)     lo     156601          0      15599          0
           rcu_sched(   10)   eth0          0          1          0          0
                sshd( 6890)   eth0          1          0          0          0
```

nssdns-watch-question
=====================
It tracing the libnss_dns.so for dns query.

```bash
[root@localhost systemtap-toolkit]# ./nssdns-watch-question -l /usr/lib64/libnss_dns.so. -t 100000
WARNING: Tracing libnss_dns(/usr/lib64/libnss_dns.so.2) for pid:0

curl(11786): www.google.com 57994us
curl(11788): www.facebook.com 57406us
curl(11790): www.github.com 4203477us
```


phpfpm-watch-req
================
It tracing phpfpm request

```bash
# ./phpfpm-watch-req -l /opt/php/sbin/php-fpm
WARNING: Tracing php-fpm for pid(0)
php-fpm(9665) GET /index.php?&123123=123&f=q (208us)
php-fpm(9665) GET /index.php?&123123=123&f=q (172us)
php-fpm(9665) GET /index.php?&123123=123&f=q (154us)
php-fpm(9665) GET /index.php?&123123=123&f=q (151us)
```

swoole-redis-watch
==================
It tracing swoole-redis write and read subroutine

```bash
./swoole-redis-watch -l /opt/php/lib/php/extensions/no-debug-non-zts-20131226/swoole.so -t 100000
WARNING: Tracing swoole.so(/opt/php/lib/php/extensions/no-debug-non-zts-20131226/swoole.so) for pid:0
php(25927) is writing for 10.200.175.90:6379 to size(41) *3
$4
hGet
$4
step
$10
attachment
10.200.175.90:6379 get reply: integer:710263830
php(29486) is writing for 10.200.175.90:6379 to size(41) *3
$4
hGet
$4
step
$10
attachment
10.200.175.90:6379 get reply: integer:709720993
```
