---
theme: default
background: null
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
title: 麒麟软件实训答辩
---

# 麒麟软件实训答辩

2022 年 6 月 20 日

---
layout: two-cols
---

# 1. Linux 基础知识

<div />

1.1 查看系统信息：`cat /etc/os-release`

![](/1-1-1.png)

1.2 基础配置
- 设置语言环境、设置键盘：`localectl`

![](/1-2-1.png)

- 设置日期和时间：`timedatectl`

::right::

1.3 管理用户和用户组

- 管理用户：`useradd`、`usermod`、`userdel`

	![](/1-3-1.png)

	![](/1-3-2.png)

- 管理用户组：
  `groupadd`、`groupmod`、`groupdel`、`gpasswd`、
  `newgrp`

	![](/1-3-3.png)

  ![](/1-2-6.png)

---
layout: two-cols
---

# 1. Linux 基础知识
&nbsp;

1.4 使用 DNF 管理软件包

- 配置 DNF、创建本地软件源仓库
- 添加、启用和禁用软件源：`dnf config-manager`

![](/1-4.png)

1.5 管理服务

- 管理系统服务
- 改变运行级别
- 关闭、暂停和休眠系统

::right::

1.6 管理进程

- 查看进程（`who`、`ps`、`top`、`kill`）
- 调度启动进程
- 挂起 / 恢复进程

![](/1-6-1.png)

![](/1-6-2.png)

![](/1-5-2.png)

---

# 2. 网络知识

<div class="grid grid-cols-2 gap-4">
<div>

2.1 OSI 7 层模型

- 物理层：通信介质的信号到数字信号

- 数据链接层：局域网之间计算机通信

- 网络层：IP 地址、路由

- 传输层：TCP 协议、UDP 协议、端口

- 会话层：控制发包的数据，会话层控制传输层三次握手

- 表示层：文件格式

- 应用层：应用程序使用

</div>
<div>

2.2 TCP、IP 协议

- 物理层、数据链接层、网络层、传输层、应用层

- 源 mac 地址和目标 mac 地址、源 IP 地址和目标 IP 地址、源端口和目标端口

2.3 ARP 协议和 VLAN

- Address Resolution Protocol，地址解析协议：把 IP 地址解析成 mac 地址

- 先发广播，再回应

- ARP 欺骗、广播风暴

</div>
</div>

---

# 2. 网络知识

<div class="grid grid-cols-2 gap-4">
<div>

2.4 tcpdump 抓包

- `-i` 指定网卡
- 默认抓 tcp，可以通过 `udp` 指定监听 udp、`icmp` 指定抓 ping 包
- `port` 指定端口
- 其他参数：`-n`、`-nn`、`-c`、`-w`、`-S`

2.5 常用的网络命令

- `nc`
- `nmap`：扫描端口
- `telnet <ip> <port>`： 连接服务器，远程登录
- `iftop` 监控外部 ip 地址的流量
- `traceroute` 追踪路由

</div>
<div>

2.6 静态路由
- 查看路由表
- 添加静态路由
- 启用内核转换参数
- 静态路由和内核转换参数的持久化存储
- 动态路由：相互学习，动态更新路由表

2.7 iptables 实现 nat 上网

- iptables 的五链：
  - PREROUTING、INPUT、FORWARD、OUTPUT、POSTROUTING
- iptables 的四表：
  - filter、nat、mangle、raw

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（1/7）

<div class="grid grid-cols-2 grid-cols-[390px,1fr] gap-4">
<div>

1. 定义 IP 和主机名变量，后续部署使用

2. 设置语言为中文、编码为 `UTF-8`

3. 利用 `firewalld` 开放端口，更新防火墙规则

</div>
<div>

```sh
LOCALMANAGEMENTIP="192.168.230.134"
LOCALOVERLAYIP="192.168.230.134"
LOCALHOSTNAME="localhost.localdomain"

echo "set zh_CN locale"
localectl set-locale LANG=zh_CN.UTF-8  #设置语言

echo "firewalld enable neutron and vxlan ports" #利用firewalld开放端口
firewall-cmd --permanent --add-port 9696/tcp #neutron
firewall-cmd --permanent --add-port 5000/tcp #keystone
firewall-cmd --permanent --add-port 5672/tcp #rabbitmq
firewall-cmd --permanent --add-port 4789/udp #vxlan
firewall-cmd --permanent --add-port 11211/tcp #memcached
firewall-cmd --permanent --add-port 11211/udp #memcached
firewall-cmd --reload #更新防火墙规则

```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（2/7）

<div class="grid grid-cols-2 grid-cols-[300px,1fr] gap-4">
<div>

4. 配置 MySQL，启动 `mariadb.service`

5. 启动 `rabbitmqp-server`，增加新用户名和密码，给用户赋予配置、读、写权限
	> 此处发生错误，在 `/etc/rabbitmq/rabbitmq-env.conf` 文件中添加 `NODENAME=rabbit@localhost`，重启 `rabbitmq-server` 解决。

</div>
<div>

```sh
echo "start to install mysql and rabbitmq" #下载mysql和rabbitmq

cat > /etc/my.cnf.d/openstack.cnf << EOF
[mysqld]
bind-address = $LOCALMANAGEMENTIP #mysql绑定地址
default-storage-engine = innodb #默认储存引擎
innodb_file_per_table = on #innodb每个表单独保存
max_connections = 4096 #最大连接数
collation-server = utf8_general_ci
character-set-server = utf8 #服务器字符集
EOF

systemctl enable mariadb.service
systemctl start mariadb.service #启动和自启动mariadb.service

systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service #启动和自启动rabbitmq-server.service
rabbitmqctl add_user openstack openstack #增加新用户和密码
rabbitmqctl set_permissions openstack ".*" ".*" ".*" #给用户赋予配置、读、写权限
systemctl restart rabbitmq-server.service #重新启动rabbitmq-server.service
```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（3/7）

<div class="grid grid-cols-2 grid-cols-[240px,1fr] gap-4">
<div>

6. 将 `memcached` 监听地址改为所有 IP，启动服务

7. MySQL 创建 `keystone` 和 `neutron` 数据库

</div>
<div>

```sh
echo "setup memcached"
sed -i '/OPTIONS/c\OPTIONS="-l 0.0.0.0"' /etc/sysconfig/memcached #memcached监听地址改为所有IP
systemctl enable memcached.service
systemctl start memcached.service #启动和自启动memcached.service

echo "***start to create table keystone and neutron***"
mysql -u root -p <<EOF 2>/dev/null
    CREATE DATABASE keystone;
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystone';
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystone';
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'$LOCALHOSTNAME' IDENTIFIED BY 'keystone';
    CREATE DATABASE neutron;
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutron';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutron';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'$LOCALHOSTNAME' IDENTIFIED BY 'neutron';
EOF
```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（4/7）

<div class="grid grid-cols-2 grid-cols-[300px,1fr] gap-4">
<div>

8. 更新 `keystone` 配置文件，设置文件所有权，设置管理 IP，初始化数据库信息和 `Fernet` 密钥数据库，并设置密码和节点

9.  设置管理 IP，配置 HTTP 配置文件，启动服务

</div>
<div>

```sh
echo "***start to set keystone*****"

rm -rf /etc/keystone/keystone.conf
cp ./keystone.conf /etc/keystone/keystone.conf #更新keystone配置文件
chown root:keystone /etc/keystone/keystone.conf #设置文件所有权
sed -i "s/172.30.20.211/${LOCALMANAGEMENTIP}/g" /etc/keystone/keystone.conf #设置管理ip

su -s /bin/sh -c "keystone-manage db_sync" keystone #初始化数据库信息
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone #初始化Fernet密钥存储库
keystone-manage bootstrap --bootstrap-password admin \ #设置密码和节点
    --bootstrap-admin-url http://${LOCALMANAGEMENTIP}:5000/v3/ \
    --bootstrap-internal-url http://${LOCALMANAGEMENTIP}:5000/v3/ \
    --bootstrap-public-url http://${LOCALMANAGEMENTIP}:5000/v3/ \
    --bootstrap-region-id RegionOne

echo "ServerName ${LOCALMANAGEMENTIP}" >> /etc/httpd/conf/httpd.conf #设置管理ip
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/ #配置http配置文件
systemctl enable httpd.service
systemctl start httpd.service #启动和自启动httpd.service

```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（5/7）

<div class="grid grid-cols-2 grid-cols-[220px,1fr] gap-4">
<div>

10. 配置用户名、密码、项目名等信息

11. 获取 openstack token 并创建 service 项目

12. 创建 neutron 用户，加入 service 项目、添加 admin 权限，创建 network 服务

</div>
<div>

```sh
cat >> ~/.bashrc << EOF
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://${LOCALMANAGEMENTIP}:5000/v3
export OS_IDENTITY_API_VERSION=3
EOF

source ~/.bashrc
openstack token issue #获取token
openstack project create --domain default --description "Service Project" service #创建service项目

echo "****start to set neutron*****"
openstack user create --domain default --password neutron neutron #创建neutron用户
openstack role add --project service --user neutron admin #将neutron用户添加入service项目并拥有admin权限
openstack service create --name neutron --description "OpenStack Networking" network #创建network服务
openstack endpoint create --region RegionOne network public http://${LOCALMANAGEMENTIP}:9696 #添加三个endpoint
openstack endpoint create --region RegionOne network internal http://${LOCALMANAGEMENTIP}:9696
openstack endpoint create --region RegionOne network admin http://${LOCALMANAGEMENTIP}:9696

```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（6/7）

<div class="grid grid-cols-2 grid-cols-[240px,1fr] gap-4">
<div>


13. 更新 neutron 配置文件，设置文件所有权和管理 IP

14. 更新 ml2 配置文件和所有权

15. 更新 openvswitch_agent 配置文件、所有权、业务 IP

16. 更新 l3_agent 配置文件和所有权


</div>
<div>

```sh
rm -rf /etc/neutron/neutron.conf
cp ./neutron.conf /etc/neutron/neutron.conf #更新neutron配置文件
chown root:neutron /etc/neutron/neutron.conf #设置文件所有权
sed -i "s/172.30.20.211/${LOCALMANAGEMENTIP}/g" /etc/neutron/neutron.conf #设置管理ip

rm -rf /etc/neutron/plugins/ml2/ml2_conf.ini
cp ./ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini
chown root:neutron /etc/neutron/plugins/ml2/ml2_conf.ini #更新ml2配置文件和所有权

rm -rf /etc/neutron/plugins/ml2/openvswitch_agent.ini
cp ./openvswitch_agent.ini /etc/neutron/plugins/ml2/openvswitch_agent.ini
chown root:neutron /etc/neutron/plugins/ml2/openvswitch_agent.ini
sed -i "s/172.30.20.211/${LOCALOVERLAYIP}/g" /etc/neutron/plugins/ml2/openvswitch_agent.ini #更新openvswitch_agent配置文件、所有权、业务ip

rm -rf /etc/neutron/l3_agent.ini
cp ./l3_agent.ini /etc/neutron/l3_agent.ini
chown root:neutron /etc/neutron/l3_agent.ini #更新l3_agent配置文件和所有权
```

</div>
</div>

---

# 3. 虚拟网络安装配置
Neutron 的脚本部署（7/7）

17. 更新 dhcp_agent 配置文件和所有权

18. 更新 ml2_conf 配置文件，同步数据库

18. 启动 neutron 服务

```sh
rm -rf /etc/neutron/dhcp_agent.ini
cp ./dhcp_agent.ini /etc/neutron/dhcp_agent.ini
chown root:neutron /etc/neutron/dhcp_agent.ini #更新l3_agent配置文件和所有权
pkill dnsmasq #杀死dnsmasq进程

ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini #更新ml2_conf配置文件
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron #同步数据库

systemctl enable neutron-server.service openvswitch.service neutron-dhcp-agent.service neutron-l3-agent.service neutron-openvswitch-agent.service
systemctl start neutron-server.service openvswitch.service neutron-dhcp-agent.service neutron-l3-agent.service neutron-openvswitch-agent.service #启动neutron服务
```

---

# 3. 虚拟网络安装配置

<div />

**验证网络操作**：使用 `openstack network agent list` 命令列出代理以验证启动 neutron 代理是否成功：
![](/agent-list.png)

**Neutron 创建网络和子网**：使用 `neutron net-create net1` 命令创建名为 `net1` 的网络，使用 `neutron subnet-create net1 192.168.2.0/24 --name subnet1` 命令创建名为 `subnet1`、IP 为 `192.168.0.2/24` 的子网：
![](/create-result.png)

---

# 4. 深入理解 Neutron - Openstack 网络实现
### 4.1 OpenStack 网络实现方式对比

- **gre**

  跨不同网络实现二次 IP 通信，L3 上面包装 L3，封装在 IP 报文中，点对点隧道，不用变更底层网络架构重建 L2、L3 通信，实现不同 host 之间网络 guest 互通，方便 guest 迁移，支持网络数量扩大。

- **vlan**

  vlan 将局域网设备从逻辑上划分成一个个网段，从而实现虚拟工作组的数据交换技术。分隔了端口，即便在同一个交换机上，处于不同 VLAN 的端口也是不能通信的。这样一个物理的交换机可以当作多个逻辑的交换机使用；使网络更加安全，不同 VLAN 不能直接通信，杜绝了广播信息的不安全性，且便于灵活管理。

- **vxlan**

  vxlan 将虚拟机发出的数据包封装在 UDP 中，并使用物理网络的 IP/MAC 作为 outer-header 进行封装，然后在物理 IP 网上传输，到达目的地后由隧道终结点解封并将数据发送给目标虚拟机。解决了 vlan 的数量和物理网络基础设施的限制，避免了 TOR 交换机 MAC 表耗尽，满足了多租户场景。

---

# 4. 深入理解 Neutron - Openstack 网络实现

### 4.2 DVR 对不同流量的区分

为了降低网络节点的负载，同时提高可扩展性，OpenStack 自 Juno 版本开始正式引入了分布式路由（Distributed Virtual Router，DVR）特性（用户可以选择使用与否），来让计算节点自己来处理原先的大量东西向流量和非 SNAT 南北流量（有 floating IP 的 vm 跟外面的通信）。

这样网络节点只需要处理占到一部分的 SNAT （无 floating IP 的 vm 跟外面的通信）流量，大大降低了负载和整个系统对网络节点的依赖。很自然的，FWaaS 也可以跟着放到计算节点上。

典型场景：从网络的访问看，涉及到路由服务的至少是需要跨子网的访问，又包括是否是同一机器、是否是涉及到外网（东西向 vs 南北向）。DVR根据如下特征区分不同流量。

| 方向 | 同一机器                    | 不同机器          |
| --- | -------------------------- | ---------------- |
| 东西 | 本地网桥处理                 | 本地东西路由器     |
| 南北 | 本地南北路由器  floating 转发 | 网络节点 SNAT 转发 |

---

# 4. 深入理解 Neutron - Openstack 网络实现
### 4.3 OpenStack 中网络节点和计算节点的不同作用

- 网络节点：网络节点有且仅有 Neutron 服务，就是网络服务。Neutron 主要负责管理私有网段和公有网段之间的通信，同时管理虚拟机网络之间的通信以及防火墙等等。一般在部署时会部署两个以上的网络端口，分别用于与控制节点通信、同计算/存储节点通信、用于外部的虚拟机与相应的网络之间的通信。

- 计算节点：计算节点主要包含计算服务、网络服务以及监控服务。计算节点对所部署的虚拟机提供基本的网络功能支持，包括隔离不同租户的虚拟机和进行一些基本的安全策略管理。计算节点包含 Nova，Neutron，Telemeter 三个服务：
	- 基础服务 Nova：提供虚拟机的创建，运行，迁移，快照等各种围绕虚拟机的服务，并提供 API 与控制节点对接，由控制节点下发任务
	- 基础服务 Neutron：提供计算节点与网络节点之间的通信服务
	- 扩展服务 Telmeter：提供计算节点的监控代理，将虚拟机的情况反馈给控制节点，是 Centimeter 的代理服务

---

# 4. 深入理解 Neutron - Openstack 网络实现
### 4.4 HAProxy 实现负载均衡策略的方式

HAProxy 提供高可用性、负载均衡以及基于 TCP 和 HTTP 应用的代理，根据运行模式可以很简单安全的整合到当前的架构中，同时可以保护web服务器不被暴露到网络上。HAProxy 应用在七层架构中，较为灵活，支持 SSL。

Neutron 支持的负载均衡算法：
- 轮询 Round robin，最普遍的算法，每当有一个新的请求来临，负载均衡器都会按顺序选择服务器池中的一台设备来响应。
- 最少连接算法 Least connections，负载均衡器收到新的请求时，都会从当前服务器池中挑选一个当前并发连接数最少的服务器来负责。
- IP hash，负载均衡器在收到主机的连接请求后，会根据数据包的源IP地址字段的 hash 值，同时按照轮询的方式为客户端分配主机，当负载均衡器再次收到同一IP的请求时，则会按照之前的记录为客户端分配上次建立连接的主机。

---
layout: center
class: text-center
---

# 谢谢
