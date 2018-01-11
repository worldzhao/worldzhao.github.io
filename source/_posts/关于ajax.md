---
title: 初识ajax
tags: [http]
categories: http
---
在学习ajax之前,如果想要发出http请求(form表单或者输入地址回车),页面会整个刷新,极其影响用户体验.  

我的理解就是ajax可以使得开发者在自定义的事件触发下进行http请求,因为是通过js完成的,可以在页面不进行刷新的情况下达到动态更新页面内容的效果.

AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。

AJAX 不是新的编程语言，而是一种使用现有标准的新方法。

AJAX 是在不重新加载整个页面的情况下,与服务器交换数据并更新部分网页的艺术。

百度的搜索推荐就是通过ajax向服务器发出请求,极少量的数据信息想要传递给服务器没必要刷新整个页面,ajax就是实现在不进行页面刷新的情况下向服务器发出http请求.

如何实现?
通过浏览器端的js帮我们预定义的一个异步对象来完成的XMLHttpRequest. 

XMLHttpRequest 是 AJAX 的基础。

发送ajax请求需要5步

1. 创建异步对象  XMLHttpRequest

XMLHttpRequest(兼容问题)  

2. 设置请求的url等参数open

3. 发送请求(请求报文,http请求)send
思考:为什么没有get方法
因为网络请求是耗时的,只有通过某个事件的发生来获取信息,比如请求状态的改变
如果存在get方法,什么时候调用?onreadystatechange?那和标准流程又有什么区别.

4. 注册事件(需要判断请求状态) onreadystatechange

onreadystatechange 事件
当请求被发送到服务器时，我们需要执行一些基于响应的任务。

每当 readyState 改变时，就会触发 onreadystatechange 事件。

readyState 属性存有 XMLHttpRequest 的状态信息。

onreadystatechange 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。 
readyState 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。

0: 请求未初始化 
1: 服务器连接已建立 
2: 请求已接收 
3: 请求处理中 
4: 请求已完成，且响应已就绪 
 
status 200: "OK"

404: 未找到页面

在 onreadystatechange 事件中，我们规定当服务器**响应已做好被处理的准备时**所执行的任务。

当 readyState 等于 4 且状态为 200 时，表示响应已就绪：

5. 在注册的事件中获取返回的内容并修改页面的显示

数据是保存在异步对象的responseText属性中

实际操作:
* 先写.html页面 通过某种条件发出ajax请求
* 再写.php页面 处理发过来的请求
* 再回到浏览器异步对象onreadystatechange事件中 处理返回的内容


上面我们是发送请求获得数据,倘若要发送数据到服务器端怎么办?

ajax通过get方法发送数据给服务器  

如果希望通过 GET 方法发送信息，请向 URL 添加信息：  

在open方法参数url中拼接要穿给服务器的数据xxx.php?userName=jack(动态拼接)  

比如用户名注册重复检测:  

blur时触发ajax事件,需要将输入的用户名传递给后台php页面,这时不是通过表单传递的,没有submit,需要我们手动将数据传给后台,我们可以通过js获取用户输入的用户名,拼接在url后面发送请求.  

	xmlhttp.open('get','check.php?userName='+un);  
	
	xmlhttp.send();

ajax通过post方法发送数据给服务器  

如果需要像 HTML 表单那样 POST 数据，请使用 setRequestHeader() 来添加 HTTP 头。  

然后在 send() 方法中规定您希望发送的数据：  

	xmlhttp.open('post','changeStar.php');  
	
	xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");  
	
	xmlhttp.send('starName='+this.dataset['star']);  

为什么要添加http头,因为post是将参数放到http后面的  

而get传参数的方式就是通过虚拟地址传送

* 封装 ajax函数
{% codeblock lang:javascript %}
	function ajax_tool(url,method,data,success){
		//异步对象
		var xmlhttp = new XMLHttpRequest();
		//url方法,get与post需要分别处理
		if(method=='get'){
			if(data){
				url+='?';
				url+=data;
			}
			xmlhttp.open(method,url);
			xmlhttp.send();
		}else{
			xmlhttp.open(method,url);
			xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
			if(data){
				xmlhttp.send(data);
			}else{
				xmlhttp.send();
			}
		}
		
		xmlhttp.onreadystatechange=function(){
			if(xmlhttp.readyState==4&&xmlhttp.status==200){
				//console.log(xmlhttp.responseText);
				//然而工具函数并不能进行数据操作
				//没有任何一个工具可以应对千奇百怪的需求
				//所以这里我们需要将数据返回,但是return是不行的
				//添加事件响应后,工具函数已执行完毕,并没有立刻执行事件函数,并没有返回值,外面接收不到
				//使用回调函数,传入一个函数作为参数,当事件终于被触发时,可以通过这个函数来对数据直接进行操作
				success(xmlhttp.responseText);
			}
		}
	} 
 {% endcodeblock %} 
* 简化ajax函数,参数过多每次都叫记得参数顺序,可以直接传入一个对象,取得对象属性值即可.此处不做演示.
