#DVWA SQL注入LOW级别实战
##目标
利用SQL注入漏洞，获取数据库中用户账号与密码，并进行解密
##漏洞原理
后台PHP代码未对用户输入进行过滤，直接将参数拼接到SQL查询语句中，类似：SELECT  first_name,last_name FROM users WHERE user_id='$id';'由于'$id'可控，我们可以通过闭合单引号，注释余下代码，并利用'UNION SELECT'伪造查询结果，将数据回显在页面上。
##实战过程
###1.探测闭合方式
  正常输入'1',页面返回admin信息
  输入1'，页面报错500 Internal Server Error,存在SQL注入，闭合符为单引号
###2.确定回显位
  构造payload：1' UNION SELECT 1,2 #
  页面返回数字，证明“UNION SELECT”生效
###3.获取核心数据
  构造payload:1' UNION SELECT user,password FROM users #
  成功获取用户名以及密码MD5哈希值
###4.解密
  打开网站cmd5.com将哈希值复制过去解密
###踩坑
  **URL编码问题**：直接输入'-- '注释容易引发语法问题
  **解决**：使用#代替
##SQL盲注实战
### 手动探测（Burp Suite）
   1.利用 `1' AND 1=1 #` 和 `1' AND 1=2 #` 判断页面回显差异，确认布尔盲注漏洞。
   2.尝试使用 Burp Intruder 爆破字符（受限于社区版功能，需手动加载 `chars.txt` 字典）。
### 自动化攻击（SQLMap）
  **踩坑记录**：DVWA Session 极易过期，手动复制 Cookie 拼命令效率极低。
  **解决方案**：利用 Burp 抓包保存为 `/tmp/req.txt` 文件，直接使用 `sqlmap -r /tmp/req.txt` 读取请求，绕过手动拼凑 Session 的烦恼。
### 战果（一步到位的拖库）
  命令：`sqlmap -r /tmp/req.txt -D dvwa -T users --dump --batch`
  成功导出所有用户数据，并自动完成了 MD5 哈希爆破（如 `admin:password`、`gordonb:abc123`）。

  
