<h1><center>Day  1</center></h1>

## 简述 

>### 服务
>
>>1. 远程链接
>>   
>>   *  **telent** [^危险，极少使用]。
>>   *  ssh
>>   *  vnc
>>   
>>2. 存储类
>>
>>   * nfs
>>   * cifs
>>   * iscsi
>>
>>3. 应用类
>>
>>    * ftp
>>* web
>>    * database
>>* mail
>>    * dhcp
>>
>### shell脚本
>>
>### 安全
>>1. 防火墙
>>   * Firewall
>>   * iptables
>>2. selinux
>### 虚拟化
>> 1. kvm
>> 2. xen
>> 3. hyper
>> 4. 半虚拟化

## 网络链接

> ### 网卡配置[^ 一张硬件网卡可以有多个配置文件但同时生效的只能有一个]
>
> > ####  1.  NetworkManager提供的方法
> >
> >    > #####  1. 图像化配置：
> >    >
> >    >    - `nm-connection-editor`
> >    >
> >    > ##### 2.类图形化：
> >    >
> >    > * `nmtui`
> >    >
> >    > ##### 3. 命令法：
> >    >
> >    > >- `nmcli`
> >    > >- 查看网卡配置
> >    > >
> >    > >   ` nmcli device show`
> >    > >
> >    > >- 查看网卡硬件状态
> >    > >
> >    > >   `nmcli device status`
> >    > >
> >    > >- 关闭网卡
> >    > >
> >    > >   `nmcli device disconnect 网卡名`
> >    > >
> >    > >- 启动网卡
> >    > >
> >    > >   `nmcli device connect 网卡名`
> >    > >
> >    > >- 修改网卡配置
> >    > >
> >    > >> 1. 修改配置文件是否为默认
> >    > >>
> >    > >>    ` nmcli connection modify 配置文件名 connection.autoconnect  yes|no`
> >    > >>
> >    > >>    
> >    > >>
> >    > >> 2. 修改配置文件的地址获取模式
> >    > >>
> >    > >>    `nmcli connection modify 配置文件名 ipv4.method auto|manual`
> >    > >>
> >    > >> 3. 修改ip地址（修改地址后需要将地址获取模式更变为manual）
> >    > >>
> >    > >>    `nmcli connection modify 配置文件名 ipv4.address 'ip/掩码 网关'`
> >    > >>
> >    > >> 4. 修改DNS地址
> >    > >>
> >    > >>    `nmcli connection modify 配置文件名 ipv4.dns 地址`
> >    > >
> >    > >- 新增网卡配置
> >    > >
> >    > >>1. DHCP模式
> >    > >>
> >    > >>          `nmcli connection add autoconnect yes con-name eth0 ifname eno16777736 type ethernet `
> >    > >>
> >    > >>2. 静态地址模式
> >    > >>
> >    > >>        `nmcli connection add autoconnect yes|no con-name eth1(配置名为eth1) ifname eno16777736(网卡名) type ethernet ip4 192.168.11.11/24 gw4 192.168.11.254`
> >    > >
> >    > >- 删除网卡配置
> >    > >
> >    > >  `nmcli connection delete 配置名`
> >    > >
> >    > >- 查看网卡配置状态 
> >    > >
> >    > >  `nmcli connection show`
> >    > >
> >    > >- 停用一个配置
> >    > >
> >    > >  `nmcli connection down 配置文件名`
> >    > >
> >    > >- 启动一个配置
> >    > >
> >    > >  `nmcli connection up 配置文件名`
> >    > >
> >    > >- 配置完毕后执行命令重启网卡管理令配置生效
> >    > >
> >    > >  `systemctl restart network` [^Centos7,Ubuntu16.04]
> >    > >
> >    > >  ` systemctl restart NetworkManager` [^Centos8,Manjaro,Ubuntu等更新的操作系统]
> >    > >
> >    > >  `service network  restar` [^Centos6，及其他更老的操作系统]
> >
> > #### 2. 直接修改网卡配置文件
> >
> > >`cd /etc/sysconfig/network-scripts/` 其中以ifcfg-开头的文件就是网卡的配置文件
> > >
> > >1. 配置文件中常用的值
> > >
> > >   ```shell
> > >   ONBOOT=yes					#设置配置随网卡启动
> > >   BOOTPROTO=none				#地址获取方式，dhcp或者none
> > >   TYPE=Ethernet				#网卡的配置类型
> > >   DEVICE=eno16777736			#该配置文件是为哪个网卡设置的
> > >   HWADDR=00:0c:29:52:2e:de	#物理地址
> > >   NAME=eth1					#配置文件名称
> > >   IPADDR0=192.168.11.11		#IP地址（后面的是数字0）
> > >   PREFIX0=24					#子网掩码
> > >   GATEWAY0=192.168.11.254		#网关
> > >   DNS1=114.114.114.114		#dns地址
> > >   ```
> >
> > #### 3. 测试网络连通性的命令
> >
> >> ````shell
> >> ping           
> >> ping6							  #ping ipv6地址
> >> ping -c 4 192.168.1.1             #选项-c用于设置ping的次数
> >> ping -I eno16777736 192.168.1.1   #选项-I用于设置ping时使用的网络设备
> >> ````
>
> ### 网卡绑定[^使用teamed]
>
> >1. #### teamd工作模式 [^ `man teamd.conf`#可以查看各个模式的具体描述]
> >
> >>```shell
> >>broadcast	         #多网卡同时工作
> >>roundrobin 	         #多网卡轮询工作
> >>activebackup         #主备模式
> >>loadbalance          #负载均衡
> >>lacp                 #基于lacp协议的一种模型，需要交换机支持
> >>```
> >
> >2. ####  以主备模式为例，如何配置网卡绑定。
> >
> >  >```shell 
> >  >#第一步：创建team虚接口
> >  >nmcli connection add autoconnect yes type team con-name team0 ifname team0\ config '{"runner":{"name":"activebackup"}}';
> >  >#第二步：将物理接口加入team虚接口
> >  >nmcli connection add autoconnect yes type team-slave con-name team0-33 ifname\ eno33555000 master team0; 
> >  >nmcli connection add autoconnect yes type team-slave con-name team0-50 ifname\ eno50332208 master team0;
> >  >#第三步（可选）：设置team的ip地址（先配地址，再改模式）
> >  >nmcli connection modify team0 ipv4.addresses '192.168.15.111/24 192.168.15.2'
> >  >nmcli connection modify team0 ipv4.dns 114.114.114.114
> >  >nmcli connection modify team0 ipv4.method manual
> >  >#第四步：重启网络
> >  >systemctl  restart  network
> >  >
> >  >查看team的状态
> >  >teamdctl  team0  state
> >  >```
> >  >

## 防火墙

>### Firewall[^RHEl7 默认防火墙软件，iptables其也有]
>
>>1. 图形化配置
>>
>>   ```shell
>>   firewall-config        #启动图形化配置窗口
>>   配置状态介绍
>>       1. runtime：临时配置，系统重启或者防火墙重载配置后会丢失
>>       2. permanent：永久配置，系统重启或者防火墙重载后才会生效的配置
>>   ```
>>
>>2. 命令行配置
>>
>>   ```shell
>>   
>>   firewall-cmd --permanent --list-all         #查看防火墙的永久配置
>>   
>>   firewall-cmd --list-all                     #查看防火墙正在运行的配置
>>   
>>   firewall-cmd --reload                       #重载防火墙配置
>>   
>>   firewall-cmd --set-default-zone=区域名称（public或者trusted） #设置防火墙的默认zone
>>   
>>   firewall-cmd --permanent --add-service=服务名   #永久打开一个服务
>>   
>>   firewall-cmd --permanent --add-port=8080/tcp   #永久打开一个端口
>>   
>>   
>>   firewall-cmd --permanent --add-forward-port='port=1234:proto=tcp:toport=4321:toaddr=' #设置端口转发
>>   
>>   firewall-cmd --permanent --add-rich-rule 'rule family="ipv4" source address="192.168.111.0/24" service name="dhcp" reject'  #设置富规则
>>   ```

## selinux

>### selinux 概念
>
>>1. 主体
>>   - 一般为系统中的服务程序
>>2. 对象
>>   - 目录、端口等系统资源
>>3. 安全上下文
>>   - selinux通过判断安全上下文的值，来决定主体能否去访问对象
>>4. 布尔值    bool
>>   - selinux通过控制预先设置好的布尔值，来决定主机是否拥有某些能力
>
>### selinux操作
>
>>1. #### 设置selinux状态
>>
>>   1. selinux的三个状态：
>>
>>      ```shell
>>      - Enforcing #启动状态，selinux会对系统进行监控，对于非法访问会进行拦截，同时记录相关日志。
>>      - permissive #宽容状态，selinux会对系统进行监控，对于非法访问不会拦截，仅记录日志。
>>      - disabled #关闭状态，selinux完全关闭
>>      ```
>>
>>      
>>
>>   2. 查看selinux当前的状态：
>>
>>      ```shell
>>      getenforce
>>      ```
>>
>>   3. 修改selinux的状态
>>
>>      ```shell
>>      setenforce 1		#设置selinux为enforcing状态
>>      setenforce 0		#设置selinux为permissive状态
>>      ```
>>
>>      文件/etc/selinux/config中SELINUX=的值决定了下次开机后selinux的状态，如果要完全关闭selinux需要将SELINUX=的值修改为disable
>>
>>2. #### 查看与修改对象的上下文
>>
>>   >```shell
>>   >
>>   >#查看目录的上下文
>>   >ls  -Zd  目录路径       
>>   >
>>   >#修改目录的上下文
>>   >semanage fcontext -a -t samba_share_t '/var/abc(/.*)?'
>>   >restorecon -RFv /var/abc
>>   >
>>   >#查设定看系统所有的文件上下文的默认设定
>>   >semanage fcontext -l | grep /var/abc
>>   >
>>   >#修改端口的上下文
>>   >semanage port -a -t http_port_t -p 协议(如tcp) 端口号
>>   >
>>   >#查看端口的上下文
>>   >semanage port -l | grep 端口号
>>   >
>>   >```
>>   >
>>   >
>>
>>3. ### 查看与修改布尔值
>>
>>   >```shell
>>   >#查看布尔值
>>   >getsebool  -a  |  grep  关键字
>>   >
>>   >#查看布尔值的简介
>>   >semanage boolean -l | grep  关键字
>>   >
>>   >#修改布尔值,其中-P表示永久修改
>>   >setsebool -P 关键字 on|off
>>   >
>>   >```
>>   >
>>   >

## systemctl

>1. ### 启动与关闭服务
>
>  >```shell
>  >systemctl  start   服务名		#开启服务
>  >systemctl  stop    服务名		#关闭服务
>  >systemctl  status  服务名		#查看服务状态
>  >systemctl  enable  服务名		#设置服务开机自启动
>  >systemctl  disable 服务名 		#设置服务开机不自启动
>  >systemctl  restart 服务名		#重新启动服务
>  >```
>
>2. 控制系统的运行目标
>
>  >1. init时代操作系统的运行级别[^命令 `init 数字` 可以更改运行级别]
>  >   - 0  关机
>  >   - 1  单用户级别（用于系统维护或错误抢救）
>  >   - 2  命令行级别（不支持nfs）
>  >   - 3  命令行级别
>  >   - 4  保留级别
>  >   - 5  图形化级别
>  >   - 6  重启
>  >2. systemed时代的系统运行目标
>  >
>  >   - multi-user.target	  多用户目标（原命令行级别 lv3）
>  >   - graphical.target	   图形化目标（原图形化级别 lv5）
>  >   - emergency.target	救援目标（原单用户级别 lv1）
>  >
>  >3. 操作[^老版`init`命令仍兼容]
>  >
>  >   >```shell
>  >   >systemctl isolate    目标                   #切换当前运行目标
>  >   >systemctl get-default                       #查看当前系统的默认运行目标
>  >   >systemctl set-default  目标                  #修改当前系统的默认运行目标
>  >   >```





[^ 一张硬件网卡可以有多个配置文件但同时生效的只能有一个]:一张硬件网卡可以有多个配置文件但同时生效的只能有一个
[^危险，极少使用]: 危险，极少使用
[^使用teamed]:使用teamed
[^Centos8,Manjaro,Ubuntu等更新的操作系统]: Centos8,Manjaro,Ubuntu等更新的操作系统
[^Centos7,Ubuntu16.04]: Centos7,Ubuntu16.04
[^Centos6，及其他更老的操作系统]: Centos6，及其他更老的操作系统
[^ `man teamd.conf`#可以查看各个模式的具体描述]: `man teamd.conf` #可以查看各个模式的具体描述
[^命令 `init 数字` 可以更改运行级别]: `init 数字` 可以更改运行级别
[^老版`init`命令仍兼容]:`init`命令仍兼容