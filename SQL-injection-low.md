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
