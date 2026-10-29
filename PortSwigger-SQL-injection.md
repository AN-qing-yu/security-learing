#PortSwigger Web Security Academy:SQL Injection
##关卡一：SQL injection vulnerability in WHERE clause allowing retrieval of hidden data（寻找隐藏数据）
**目标**：利用SQL注入漏洞，让电商网站显示原本隐藏的商品
**漏洞发现**：点击商品分类时，URL为https://0a4f00e204f0621c825e20a500c800ce.web-security-academy.net/filter?category=Gifts猜测后端查询为SELECT * FROM products WHERE category = 'Gifts' AND released = 1
**攻击Payload**:Gifts ' OR 1=1 --
**原理**：'(单引号)：闭合后端原本字符串，使前面的查询条件完整【条件闭合】
         OR 1=1：构造绝对正确的条件。OR或者，不管分类是什么，都会打印所有数据【欺骗以便拉取所用有数据】
         --：将SQL后续所有内容注释【告诉数据库到这就可以了】
**防御**：使用参数化查询(预编译)JAVA中用PreparedStatement PHP中用PDO，避免字符串直接拼接【预判hack的预判】

##关卡二：SQL injection vulnerability allowing login bypass（允许登录绕过）
**目标**：在不知道管理员密码的前提下，以管理员administrator身份成功登录系统【拿着管理员的牌子蒙混过关】
**漏洞发现**：目标是一个登录框，后端查询大概为SELECT * FROM users WHERE username='输入' AND password='输入'
**攻击Payload**:用户名：administrator'--
                密码：随便
**原理**：利用'(单引号)闭合用户名参数
         利用--注释密码验证
         数据库只验证了用户名直接放行
**个人思考**：为什么用户名必须administrator而不是admin，原因：数据库中没有admin

##总结
红队：注意信息收集，注意URL
蓝队：预判攻击者的预判，在编写代码时使用安全性更高的
