CentOS实现家庭路由器功能

# 1 架构

- eth0连接Modem，通过PPPOE拨号联网；
- eth1连接内网交换机，提供DHCP服务；



# 2 拨号联网

参考[鸟哥的Linux私房菜 第四章连上Internet](http://cn.linux.vbird.org/linux_server/0130internet_connect_2.php)

### 2.1 安装rp-pppoe

### 2.2 设置pppoe

1. 输入电信运营商提供的账号
2. 选择拨号的网卡eth0
3. Enter the demand value: no
4. 输入DNS IP（可以使用腾讯119.29.29.29或阿里223.6.6.6）
5. 输入电信运营商提供账号的密码（输入两次）
6. USERCTRL：no
7. 先关闭防火墙
8. 设置开机启动拨号



### 2.3 检查pppoe

`shell# ifconfig ppp0`

检查是否获得类似`inet addr:111.255.69.90  P-t-P:168.95.98.254  Mask:255.255.255.255`信息输出。

1. 111.255.69.90就是拨号获得公网IP地址；
2. 168.95.98.254就是网关GatewayIP地址；
3. 即eth0已经拨号成功，并获得公网等IP信息。



# 3 设置内网

参考[CentOS做网关路由器详细实现方法](http://network.51cto.com/art/201512/500881.htm)

### 3.1 设置eth1

内网IP网段是`192.168.200.XX`，网卡eth1需要配置固定IP地址。

```shell
DEVICE=eth1
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.200.1
NETMASK=255.255.255.0
```



# 4 包转发

### 4.1 启动包转发功能

```shell
shell# vi /etc/sysctl.conf 
#添加 net.ipv4.ip_forward = 1
shell# sysctl -p (刷新执行，或者使用以下的方式)
shell# echo 1 > /proc/sys/net/ipv4/ip_forward
```



### 4.2 iptables

参考[CentOS6.5做路由器](http://blog.51cto.com/gjr0512/1767923)

```shell
shell# iptables -F  # 清除iptables
shell# iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE  # 打开NAT
shell# iptables -p INPUT ACCEPT
shell# iptables -p FORWARD  ACCEPT
shell# iptables-save  
shell# iptables-restore /etc/sysconfig/iptables
```



### 4.3 查看路由表

```shell
shell# netstat -rn
shell# route -n
```



### 4.4 测试

客户端通过网线连接CentOS的eth1接口，并设置客户端的网卡地址。

编辑`/etc/sysconfig/network-scripts/ifcfg-eth0`，添加以下内容：

```shell
IPADDR=192.168.200.100
NETMASK=255.255.255.0
GATEWAY=192.168.200.1
```

保存并重启网卡：

```shell
/net/init.d/network restart 	
```

客户端测试：

```shell
ping www.baidu.com	
```



# 5 设置DHCP

参考[鸟哥私房菜-DHCP服务器](http://cn.linux.vbird.org/linux_server/0340dhcp_2.php)

### 5.1 基础设置

```shell
shell# vim /etc/dhcp/dhcpd.conf
# 1. 整体的环境设定
ddns-update-style            none;            <==不要更新 DDNS 的设定
ignore client-updates;                        <==忽略客户端的 DNS 更新功能
default-lease-time           259200;          <==预设租约为 3 天
max-lease-time               518400;          <==最大租约为 6 天
option routers               192.168.100.1;   <==这就是预设路由
option domain-name           "centos.youlike";  <==给予一个领域名
option domain-name-servers   119.29.29.29, 223.6.6.6;
# 上面是DNS的IP设定，这个设定值会修改客户端的/etc/resolv.conf档案内容

# 2. 关于动态分配的IP
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.101 192.168.100.200;  <==分配的 IP 范围

    # 3. 关于固定的 IP 啊！
    host win7 {
        hardware ethernet    08:00:27:11:EB:C2; <==客户端网卡 MAC
        fixed-address        192.168.100.30;    <==给予固定的 IP
    }
}
# 相关的设定参数意义，请查询前一小节的介绍，或者 man dhcpd.conf
```

### 5.2 启动观察

```shell
shell# /etc/init.d/dhcpd start
shell# chkconfig dhcpd on
shell# netstat -tlunp | grep dhcp
shell# tail -n 30 /var/log/messages
shell# tail -f /var/log/messages
```

### 5.3 测试

参考[鸟哥私房菜-自动获取IP](http://cn.linux.vbird.org/linux_server/0130internet_connect_2.php#connect_auto)

客户端通过网线连接CentOS的eth1接口，并设置为动态获取IP地址：

```shell
shell# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR="XXX"
NM_CONTROLLED="no"
ONBOOT=yes
BOOTPROTO=dhcp
shell# /etc/init.d/network restart
...
Determining IP information for eth0.. [ OK ] 
```



