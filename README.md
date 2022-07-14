# netns

networknamespace notes:

1.ip netns add netns1  -- 创建名称为netns1的网络名称空间

2.ip netns exec netns1 ip link list  -- 查看名称空间内的网络设备
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

3.ip netns exec netns1 ping 127.0.0.1 -- 验证当前lo设备无法通信
  connect: Network is unreachable 

4.ip netns exec netns1 ip link set dev  lo up  -- 启动lo设备

5.ip link add veth0 type veth peer name veth1  -- 创建veth 设备对

6.ip link set veth1 netns netns1  -- 将veth1 link 到netns1中

7.ip netns exec  netns1 ifconfig veth1 10.1.1.1/24 up  --启动netns 中的veth1的虚拟网卡

8.ifconfig veth0 10.1.1.2/24 up   -- 将主机上的veth0的网卡启动

ip link add veth2 type veth peer name veth3

ip addr add 1.2.3.101/24 dev veth2
    
ip addr add 1.2.3.102/24 dev veth3

ip link set veth2 up
    
ip addr add 1.2.3.102/24 dev veth3

ip link set dev veth2  mastar br0  -- 将虚拟网卡link到网桥上
