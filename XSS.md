##XSS(Reflected)反射型实战
###步骤
   1.输入页面要求输入内容，点击提交，发现输入框下面出现'hello 你输入的内容'
   2.因为输出内容推测，后台代码应该是'echo $_GET['name']'，尝试让其出现弹框
   3.向对话框输入'<script>alert('XSS')</script>'
   4.点击提交后出现弹框
###进阶尝试
   1.将难度调为MEDIUM
   2.在对话框内输入'<script>alert('XSS')</script>'，没有出现预期弹框
   3.尝试将语句变为大小写混用，弹框出现
   4.同时思考是否可以改用其他语句，采用<img src=1 onerror=alert('XSS')>,弹框出现实验成功
##XSS（stored）存储型（危害很大直接寄生与服务器）
###步骤
   1.根据页面内容填写name:angel(你喜欢就好随便)message：'<script>alert('XSS')</script>'点击提交后出现弹框
   2.刷新页面发现弹框依旧出现，证明已经寄生于服务器
###进阶尝试
   1.换一个名字，在下面留言板输入<img src=1 onerror=alert('XSS')>
   2.出现弹框，证明XSS存储成功
##XSS(DOM)(前端JS逻辑问题)
###步骤
   1.随便选择一种语言，点击select发现浏览器地址栏出现改变
   2.尝试改写选择内容：default=English改为default=<script>alert<'XSS'></script>
   3.回车出现弹框
###进阶尝试
   1.中级难度，发现<script>被过滤了，改用URL编码也失败了
   2.采用</option></selrct><img src=1 onerror=alert(/XSS/)>
     原理：强行闭合前面原有的标签，使用<img>配合onerror代替<script>，使用alert（/XSS/）正则表达式代替alert（'xss'），避开单引号
   3.回车，出现弹框
###总结
 XSS反射与存储本质上一致，只不过一个是临时的，一个是永久，后者的危害更高，且不易发现，DOM是JS的调用问题
