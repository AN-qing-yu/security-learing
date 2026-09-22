# security-learing
记录学习网安笔记，靶场练习以及python脚本
9.22
在kali中安装DVWA靶场
1.切换到root用户，检查发现kali中没有docker,开始采用apt install docker.io -y命令下载docker
2.采用systemctl start docker,以及systemctl enable docker 开启docker,最后采用docker --version检查版本安装情况
3.采用docker pull vulnerables/web-dvwa命令下载dvwa，网络请求超时换用apt install dvwa -y下载
4.启动Apcha 和 MySQL 采用systemctl start apache2,systemctl start mariadb两条命令
5.点开浏览器发现404 Not Found,Apache找不到网页文件
    解决：ls /usr/share/dvwa查看文件在哪
         ln -s /usr/share/dvwa /var/www/html/dvwa链接网站根目录
         重启Apache systemctl start apache2
         网页显示500，运行有误
         查找问题：tail -n 20 /var/log/apach2/error.log发现数据库有问题
                  进入数据库：mysql -u root
                  建立数据库并授权：CREATE DATABASE IF NOT EXISTS dvwa;
                                  CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd';
                                  GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
                                  FLUSH PRIVILEGES;
                                  EXIT;
                  检查配置文件：grep -r "p@ssw0rd" /usr/share/dvwa/config/出现内容说明解决
6.重启apache服务：systemctl start apache2
7.成功打开
8.登录：admin,password
  权限报错：`/etc/dvwa/config` 显示 No。使用 `mkdir -p` 补全目录，`chown -R www-data:www-data` 赋予 Apache 权限
9.点击最下面的创建，跳转后点击左边Security Level改为low,电机submit
## 结果
成功登录，安全等级设为 Low，准备开始打靶。
