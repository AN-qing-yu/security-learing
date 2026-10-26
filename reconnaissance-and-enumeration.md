#红队实战基础：信息收集与网络枚举（hacklab游戏实战）
##背景完成DVWA/Pikachu等基础WEB练习，开始信息收集与网络枚举
##核心步骤
###模块一：足迹与侦察（Footprinting）
**目标**：在不发送侵入性流量的情况下，收集目标公开信息
**命令**
 1.whois megacorp.local:查域名注册信息【找公司】
 2.nslookup megacorp.local:域名对应IP地址【房间有哪些】
 3.theharvester -d megacorp.local -b google:收集员工邮箱
 4。dnsrecon -d megacorp.local:枚举DNS记录，查隐藏子域名

###模块二：网络扫描（Network Scaning）
**目标**：发现内网活着的主机以及开放端口，进行操作系统指纹识别
**命令**：
 1.nmap -sn 10.10.10.0/24:ping扫描，快速发现该网段存活主机【看看什么地方有人】
 2.nmap -sS 10.10.10.5:SYN半开扫描，隐蔽且快速探测目标开放端口【敲门，看看谁开门】
 3.nmap -O 10.10.10.5:基于TCP/IP协议猜测操作系统【看看是什么人】
 4.hping3 -S -p 80 -c 3 10.10.10.5:自定义数据包，绕过Ping防火墙，探测WEB端口连通性【写信看看有没有人回】

###模块三：服务枚举（Enumeration）
**目标**：针对已开放的SMB,SNMP服务，深挖用户，共享目录和系统进程信息
**命令*：
 1.enum4linux -a 10.10.10.5:爆出系统用户列表【住着什么人】
 2.smbclient -L //10.10.10.5 -N:利用匿名登录访问敏感信息【有什么好东西在家里】
 3.snmpwalk -v2c -c public 10.10.10.5:利用默认public直接拉取系统信息以及数据库【清单汇总】

 ##总结
 明白第一步该怎么做，收集什么，什么有用，什么可以备用
