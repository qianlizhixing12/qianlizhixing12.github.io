---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二周
Date: 2020-04-27
Tags: glib, CORS
---

## Algorithm

leetcode 25  K个一组翻转链表
- [原题](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/25.c)

## Review

> https://developer.gnome.org/glib/stable/
>
> https://developer.ibm.com/tutorials/l-glib/

glib是比较好的第三方C库，是gnome的基础库，提供了常用的数据结构和算法：

- Memory chunks
- Doubly-linked lists
- Singly-linked lists
- Hash tables
- Strings (which can grow dynamically)
- String chunks (groups of strings)
- Arrays (which can grow in size as elements are added)
- Balanced binary trees
- N-ary trees
- Quarks (a two-way association of a string and a unique integer identifier)
- Keyed data lists (lists of data elements accessible by a string or integer id)
- Relations and tuples (tables of data which can be indexed on any number of fields)
- Caches

glib的queue是使用链表实现的，不是常用的数组。

## Tip

> https://leetcode-cn.com/problems/valid-parentheses/solution/valid-parentheses-fu-zhu-zhan-fa-by-jin407891080/

```python3
class Solution:
    def isValid(self, s: str) -> bool:
        dic = {'{': '}',  '[': ']', '(': ')', '?': '?'}
        stack = ['?']
        for c in s:
            if c in dic: stack.append(c)
            elif dic[stack.pop()] != c: return False 
        return len(stack) == 1
```

栈操作技巧

- 避免空栈时pop弹出操作异常，初始化push一个业务无关的元素，注意后面逻辑控制

## Share

> https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
>
> https://javascript.info/fetch-crossorigin
>
> https://api.jquery.com/jQuery.ajax/https://www.yogihosting.com/jquery-jsonp-example/

### 同源策略

一个页面只能获取这个页面相同源或者相同域的数据。

同源，同域：协议、域名、端口号都必须完全相同。

#### 可用性

- 允许通过标签访问第三方框架，图片等资源(<script><img><iframe><link>等带有src属性标签)
- 允许跨域写操作：表单提交(csrf攻击)重定向请求

#### 安全性

- cookie，localstorage和indexdb无法读取
- dom无法获得(防止跨域篡改)
- Ajax请求不能发送

### 合法跨域

CORS:Cross-Origin Resource Sharing

如果站点A允许站点B的脚本访问其资源，必须在HTTP响应中显式的告诉浏览器

- 访问站点A的请求，浏览器应告知该请求来自站点B
- 站点A的响应中，应明确那些跨域请求是被允许的，以便浏览器筛选请求

#### 简单请求

Origin，Access-Control-Allow-Origin

#### 复杂请求

Access-Control-Allow-Methods，Access-Control-Allow-Headers，Access-Control-Max-Age

### JSONP

JSON是一种数据交换格式，而JSONP是一种依靠开发人员的聪明才智创造出的一种非官方跨域数据交互协议。一个是描述信息的格式，一个是信息传递双方约定的方法。

#### 跨域执行js

跨域js文件中的代码浏览器是可以执行的，服务器a.com有个a.js文件;服务器b.com下有个b.html页面。

页面会弹出一个提示窗体，显示跨域调用成功。

```html
//a.js
alert('a.js');

//b.html
<html>
	<head>
		<title></title>
		<script type="text/javascript" src="http://a.com/a.js"></script>
	</head>
</html>
```

#### 跨域js传递数据

在服务器b.com下b.html页面定义一个函数，然后在服务器a.com下a.js中传入数据进行调用。

页面会弹出一个提示窗体，显示本地回调函数被跨域的远程js调用成功，还接收了远程js带来的数据。

```html
//a.js
localHandler({"result":"返回数据"})

//b.html
<html>
	<head>
		<script type="text/javascript">
		var localHandler = function(data){
			alert('回调函数，可以被跨域的a.js文件调用，远程js返回的数据是：' + data.result);
		};
		</script>
		<script type="text/javascript" src="http://a.com/a.js"></script>
	</head>
</html>
```

#### 跨域js传递回调

问题：怎么让远程js知道它应该调用的本地函数叫什么名字？？？

答案：请求js文件时将回调函数当参数传递，服务端动态生成js脚本(利用本地回调函数运行在浏览器本地传递数据)。

```html
//http://a.com/user/getUserInfo.js
callbackFun({
	userid: 007,
	username: 'test'
});

//b.html
<html>
	<head>
		<title></title>
		<script type="text/javascript">
		var callbackFun = function(data){
			return data;
		};
		var url = "http://a.com/user/getUserInfo.js?userid=007&callback=callbackFun";
		// 创建script标签，设置其属性
		var script = document.createElement('script');
		script.setAttribute('src', url);
		// 把script标签加入head，此时调用开始
		document.getElementsByTagName('head')[0].appendChild(script); 
		</script>
	</head>
</html>
```

#### jQuery封装jsonp

```html
<html>
 <head>
	 <title></title>
	  <script type="text/javascript" src=jquery.min.js"></script>
	  <script type="text/javascript">
	 jQuery(document).ready(function(){$.ajax({
             url: "http://a.com/user/getUserInfo.js?userid=007",
			 type: "get",
			 dataType: "jsonp",
			 jsonpCallback:"callbackFun",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
			 success: function(json){
				 alert('查询到用户信息：id ' + json.userid + ' name ' + json.username);
			 },
			 error: function(){
				 alert('fail');
			 }
		 });
	 });
	 </script>
   </head>
 </html>
```