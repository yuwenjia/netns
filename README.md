# netns

2. 2   操练：

2.2.1  查看network namespace

查看network namespace
[root@host-005056b23337 ~]# ip netns list
cni-301e17d2-4e31-e494-d8f3-10ac7be04d86
cni-f10ae380-ee70-a027-7c88-ed898473d76d

2.2.2  创建network namespace

创建network namespace
[root@host-005056b23337 ~]# ip netns add testA
[root@host-005056b23337 ~]# ip netns list
testA


2.2.3  删除network namespace

删除network namespace
[root@host-005056b23337 ~]# ip netns delete testA


2.2.4  进入network namespace  

[root@host-005056b23337 ~]# ip netns exec testA ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 
 
[root@host-005056b23337 ~]# ip netns exec testA ip link set lo up
[root@host-005056b23337 ~]# ip netns exec testA ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
 
[root@host-005056b23337 ~]# ip netns exec testA ip route show

    2.3  实验：

      做一个实验，目的root network space 和自创建network space的隔离

 2.3.1   查看root network namespace 网卡信息

查看root network namespace
[root@localhost2 bin]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:b2:93:be brd ff:ff:ff:ff:ff:ff
    inet 10.131.136.238/20 brd 10.131.143.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fd00:212:34ff:fe50:250:56ff:feb2:93be/64 scope global noprefixroute dynamic
       valid_lft 2591797sec preferred_lft 604597sec
    inet6 2200::250:56ff:feb2:93be/64 scope global deprecated noprefixroute dynamic
       valid_lft 436243sec preferred_lft 0sec
    inet6 fe80::250:56ff:feb2:93be/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

2.3.2  将root network namespace 网卡 link 到 testA 的命名空间中

link testA
[root@localhost2 ~]# ip link set dev eth0 netns testA
[root@host-005056b23337 ~]# ip netns exec testA ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
 eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
 
 
[root@host-005056b23337 ~]# ip netns exec testA ip link  set eth0 up
[root@host-005056b23337 ~]# ip netns exec testA ip link  addr  add  10.131.136.238/20 dev eth0

2.3.3  验证隔离性

验证隔离性
[root@localhost2 ~]# ip netns exec  testA  ping -c 2 10.131.136.238
connect: Network is unreachable
至此Network Namespace 的隔离性验证完毕
    
3.   Veth pair 

3.1  基本概念

      veth是虚拟以太网卡（Virtual Ethernet）的缩写。veth设备总是成对的，因此我们称之为veth pair。veth pair一端发送的数据会在另外一端接收，非常像Linux的双向管道。根据这一特性，veth pair常被用于跨 network namespace之间的通信，即分别将veth pair的两端放在不同的namespace里

3.2   操练

3.2 1 创建veth pair 

创建veth pair
[root@host-005056b23337 ~]# ip link add vethA type veth peer name vethB
[root@host-005056b23337 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1600 qdisc fq_codel master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:33:37 brd ff:ff:ff:ff:ff:ff
9: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:8e:21 brd ff:ff:ff:ff:ff:ff
10: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:62:19 brd ff:ff:ff:ff:ff:ff
11: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:ae:1b brd ff:ff:ff:ff:ff:ff
12: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:d7:1f brd ff:ff:ff:ff:ff:ff
13: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:08:78 brd ff:ff:ff:ff:ff:ff
14: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d6:5a:7e:b0:09:19 brd ff:ff:ff:ff:ff:ff
15: management: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d6:23:1b:7d:dc:80 brd ff:ff:ff:ff:ff:ff
 
16: vethB@vethA: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a2:2e:11:ce:d6:7f brd ff:ff:ff:ff:ff:ff
17: vethA@vethB: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1a:49:85:72:cc:ff brd ff:ff:ff:ff:ff:ff
 
    3.2.2  分别创建两个netwrork 

创建两个network
[root@host-005056b23337 ~]# ip netns add netA
[root@host-005056b23337 ~]# ip netns add netB
3.2.3 分别将接口 vethA 加入到 netA，将接口 vethB 加入到 netB

分别加入
[root@host-005056b23337 ~]# ip link set vethA netns netA
[root@host-005056b23337 ~]# ip link set vethB netns netB
[root@host-005056b23337 ~]# ip netns exec netA ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
17: vethA@if16: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1a:49:85:72:cc:ff brd ff:ff:ff:ff:ff:ff link-netns netB
[root@host-005056b23337 ~]# ip netns exec netB ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
16: vethB@if17: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether a2:2e:11:ce:d6:7f brd ff:ff:ff:ff:ff:ff link-netns netA


3.2.4 查看root network 

消失
[root@host-005056b23337 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1600 qdisc fq_codel master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:33:37 brd ff:ff:ff:ff:ff:ff
9: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:8e:21 brd ff:ff:ff:ff:ff:ff
10: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:62:19 brd ff:ff:ff:ff:ff:ff
11: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:ae:1b brd ff:ff:ff:ff:ff:ff
12: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:d7:1f brd ff:ff:ff:ff:ff:ff
13: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b2:08:78 brd ff:ff:ff:ff:ff:ff
14: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d6:5a:7e:b0:09:19 brd ff:ff:ff:ff:ff:ff
15: management: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d6:23:1b:7d:dc:80 brd ff:ff:ff:ff:ff:ff
    
    
    3.2.3  启动网口并设置IP

启动网口并设置IP
[root@host-005056b23337 ~]# ip netns exec netA ip link set vethA up
[root@host-005056b23337 ~]# ip netns exec netB ip link set vethB up
[root@host-005056b23337 ~]# ip netns exec netA ip addr add 172.168.200.1/24 dev vethA
[root@host-005056b23337 ~]# ip netns exec netB ip addr add 172.168.200.2/24 dev vethB
[root@host-005056b23337 ~]# ip netns exec netA ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
17: vethA@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 1a:49:85:72:cc:ff brd ff:ff:ff:ff:ff:ff link-netns netB
    inet 172.168.200.1/24 scope global vethA
       valid_lft forever preferred_lft forever
    inet6 fe80::1849:85ff:fe72:ccff/64 scope link
       valid_lft forever preferred_lft forever
[root@host-005056b23337 ~]# ip netns exec netB ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
16: vethB@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether a2:2e:11:ce:d6:7f brd ff:ff:ff:ff:ff:ff link-netns netA
    inet 172.168.200.2/24 scope global vethB
       valid_lft forever preferred_lft forever
    inet6 fe80::a02e:11ff:fece:d67f/64 scope link
       valid_lft forever preferred_lft forever
3.2.3  连通性

连通性
[root@host-005056b23337 ~]# ip netns exec netB ping 172.168.200.1
PING 172.168.200.1 (172.168.200.1) 56(84) bytes of data.
64 bytes from 172.168.200.1: icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from 172.168.200.1: icmp_seq=2 ttl=64 time=0.024 ms
64 bytes from 172.168.200.1: icmp_seq=3 ttl=64 time=0.025 ms

    
    4.  对应到k8s 集群中 

   4.1  创建一个名称为pod  yaml ，查看root network namespace 

1
28: veth54dfef0c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether da:de:3c:fa:49:0e brd ff:ff:ff:ff:ff:ff link-netnsid 6
4.2 查看容器network namespace

进入容器
[ root@nginx-kkkk12:/etc/nginx ]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
    link/ether 3e:4a:e1:5d:41:7f brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.4/24 brd 10.244.0.255 scope global eth0
       valid_lft forever preferred_lft forever
4.3  查看cni0 

查看cni0
[root@localhost1 manifests]# brctl show
bridge name     bridge id               STP enabled     interfaces
cni0            8000.66b2eff50909       no              veth54dfef0c
