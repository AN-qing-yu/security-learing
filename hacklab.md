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
 明白第一步信息收集该怎么做，收集什么，什么有用，什么可以备用

 ###模块四：漏洞分析（Vulnerability Analysis）
 **目标**：从扫描结果中分析高危漏洞，并寻找可利用代码
 **命令**
  1.nikto -h 10.10.10.5:WEB漏洞扫描器，自动探测是否采用老旧服务器版本，有没有危险配置文件，有没有敏感目录暴露【检测年久失修配件，可以拆卸进去】
  2.searchsploit CVE-2021-41773:漏洞检索工具，看之前有没有一样的漏洞【寻找现有钥匙】
  3.confirm CVE-2021-41773:运行漏洞脚本，确认可以渗透【钥匙是否能开门】

###模块五：系统破解与哈希破解（System Hacking）
**目标**：获取密码并离线破解
**命令**：
1.getuid:获取UID【是谁】
2.hashdump:抓取密码哈希【拿到密码线索】
3.john --wordlist=rockyou.txt hashes.txt:john开膛手杰克，密码破解神器【破解密码】

##模块六：恶意软件分析（Malware Analysis）
**目标**：对可疑软件进行分析
**命令**
 1.file suspicious.exe:查看文件什么格式【东西的真面目】
 2.strings suspicious.exe | head:抽取文件内容显示隐藏文本【情报获取】
 3.sha256sum suspicious.exe:计算哈希值【提取指纹】
 4.vt-lookup <hash>：将拿到的指纹去查询【全网通缉】
 ##总结
 红队：找到漏洞，进行搜索看看有没有类似的，借鉴前人工具，离线破解避免痕迹泄露
 蓝队：发现可疑软件，进行内容提取，下达通缉令

###模块七：局域网中间人攻击（HITM）
**武器库**：bettercap、tcpdump
**攻击链**：
 1.ifconfig eth0:确认攻击方IP，看看处于什么网段【战前准备，查看武器以及埋伏地点】
 2.bettercap -iface eth0:bettercap局域网攻击武器，伪装MAC作为中间人截胡数据【诱骗上当】
 3.tcpdump -i eth0 -A port 80:tcpdump抓包工具
                               -i eth0监听网卡
                               -A 以ASCII码显示数据内容
                                port 80抓取80端口【获取数据】
###模块八：社会工程学钓鱼（SET）
**武器库**：setoolkit
**攻击链**：
 1.setoolkit&set-pick2:社会工程学包，set-pick2"万能钥匙"【在工具库里制造一把万能钥匙】
 2.clone https://portal.megacorp.local/login:网页克隆【一个假场景】
 3.send-mail staff@megacorp.local:伪装发送邮件【一个钓鱼邮件】
##总结
利用人性的弱点，诱导点击导致文件泄露

###模块九：拒绝服务与缓解（DoS）
**目标**：防守恶意攻击
**命令**：
 1.hping3 --fiood -S -p 80 10.10.10.5:--flood:洪水模式，疯狂发包
                                      -S发送SYN包（第一次请求握手）【红队攻击，不断打电话，导致占线】
 2.netstat-target:察看作战现场【看看什么情况】
 3.mitigate syn:企业防火墙【解决方案】

##总结
红队最大的攻击目标实际是人，利用人性的弱点诱导上当，导致核心泄露
蓝队不仅仅要死守防火墙还要注意人在里面参与的部分

###模块十：OWASP ZAP抓包演练（ZAP）
**目标**：了解此工具使用方法
**命令**
 1.zap-cli start:启动工具
 2.intercept on：开启拦截
 3.edit-cookie isAdmin=true：篡改通行证
 4.forward:放行

###模块十一：数据取证（从镜像中恢复证据）
**目标**：找到被删除的数据
**命令**
 1.mmls disk.img:打开硬盘看看里面有什么
 2。foremost -i disk.img -o recovered:恢复文件【招魂】
 3。sha256sum disk.img:提取指纹【锁定证据】

##总结
使用hacklab沉浸式体验作为一名安全工作人员的工作流程，收集信息（找到目标，寻找弱点），网络扫描（查找开放主机以及端口），渗透进入（找到漏洞潜入他家埋伏），找到核心数据（离线破解，如果你不想被抓）
学会使用工具（一个合格的安全人员应该有一个强大个武器库），社会工程学（人就是一个很大的BUG），可以从接收方拦截，也可以直接拦截源头
蓝队要使用好防火墙同时也要学会“招魂”（从被删除文件提取线索）

