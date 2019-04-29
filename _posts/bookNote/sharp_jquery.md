---
title: 锋利的jQuery
date: 2019/4/29 9:18:46 PM
tags: [jQuery]
categories: [读书笔记,jQuery]

---

### 一、了解 jQuery

* 1. $ 是jQuery  的简写形式    例如：

  ``` javascript
  $('#foo') === jQuery('#foo')
  ```

* 2. jQuery对象和 DOM对象相互转换

  ``` javascript
  var $variable = jQuery 对象
  var variable = DOM 对象
  ```

  * a.jQuery 对象是类似数组的对象，可通过[index]的方法得到相应的DOM对象

  ``` javascript
  var $cr = $('#cr');     //jQuery 对象
  var cr = $cr[0];        // DOM 对象
  ```

  * b. get(index)方法得到DOM对象

  ``` javascript
  var $cr = $('#cr');     //jQuery 对象
  var cr = $cr.get(0);    // DOM 对象
  ```

  * c. DOM对象转成 jQuery 对象: 用$()把DOM对象包装起来，便得到jQuery对象

  ``` javascript
    var cr = document.getElementById('cr');    // DOM对象
    var $cr = $(cr);                           // jQuery对象
  ```

  > jQuery 对象不可以使用DOM中的方法

  * d. 检查某个元素在网页上是否存在

  ``` javascript
    //jQuery判断：应根据获取到元素的长度来判断
    if($('#tt').length > 0){
    // do something
    }
    //DOM对象判断：
    if($('#tt')[0]){
    // do something
    }
  ```

### 二、选择器

1. 选择器中含有特殊字符，使用转义符转义

   ``` html
   <div id="id#b">123</div>
   <div id="id[1]">456</div>
   ```

   正确的获取元素写法如下：

   ``` javascript
   $('#id\\#b');       // 转义特殊字符"#"
   $('#id\\[1\\]');    // 转义特殊字符"[ ]"
   ```

2. .is(selector):

   * 根据选择器检查当前匹配元素集合，如果存在至少一个匹配元素返回 true

   * selector: 字符串值，包含匹配元素的选择器表达式


### 三、jQuery 中的DOM操作

1. 删除节点

   * remove() 方法： 从DOM中删除所有匹配的元素，传入的参数用于根据jQuery 表达式来筛选元素

     ```  javascript 
     $('ul li:eq(1)').remove() //获取第2个<li>元素节点，从网页中删除
     $('ul li').remove('li[title!=菠萝]') // 将<li>元素中属性title不等于"菠萝"的<li>元素删除
     ```

     

   * detach() 方法：从DOM中去掉匹配元素，不会把匹配元素从jQuery对象中删除。与remove()不同的是绑定的事件，附加数据会保留下来

     ``` javascript
     $('ul li').click(function(){
         alert($(this).html());
     })
     var $li = $('ul li:eq(1)').detach(); // 删除元素
     $li.appendTo('ul'); // 重新追加此元素，发现它之前绑定的事件还在
     ```

     

   * empty() 方法:  不是删除节点，是清空节点

2. 包裹节点
     ``` html
     <strong title="选择你喜欢的水果">你最喜欢的水果是？</strong>
     <strong title="选择你喜欢的花">你最喜欢的花是？</strong>
     ```

     * wrap(): 将所有元素进行单独包裹

     ``` javascript
     $('strong').wrap('<b></b>')
     ```

     会得到：

     ``` html
     <b><strong title="选择你喜欢的水果">你最喜欢的水果是？</strong></b>
     <b><strong title="选择你喜欢的花">你最喜欢的花是？</strong></b>
     ```

     * wrapAll(): 将所有匹配的元素用一个元素来包裹

     ``` javascript
     $('strong').wrapAll('<b></b>')
     ```
    
     会得到：   
     ``` html
     <b>
     	<strong title="选择你喜欢的水果">你最喜欢的水果是？</strong>
     	<strong title="选择你喜欢的花">你最喜欢的花是？</strong>
     </b>
     ```


3. 设置和获取值

     * val()方法 ： 能使select（下拉列表框）、checkbox（多选框）、radio（单选框）相应的选项被选中

       ``` html
       <!-- 单项被选中 -->
       <select id="single">
           <option>选择1号</option>
           <option>选择2号</option>
       </select>
       <!-- 多项被选中 -->
       <select id="multiple" multiple="multiple">
           <option selected="selected">选择1号</option>
           <option>选择2号</option>
           <option>选择3号</option>
       </select>
       <!-- 多选框 -->
       <input type="checkbox" value="check1"/>	多选1
       <input type="checkbox" value="check2"/>	多选2
       <input type="checkbox" value="check3"/>	多选3
       <!-- 单选框 -->
       <input type="radio" value="radio1" name="single"> 单选1
       <input type="radio" value="radio2" name="single"> 单选2

       ```

       ``` javascript
       $('#single').val('选择2号');
       $('#multiple').val(['选择2号','选择3号']);    //以数组的形式赋值
       $(':checkbox').val(['check2','check3']);
       $(':radio').val(['radio2']);
       ```

### 四、jQuery 中的事件和动画

1.加载DOM

a. window.onload:

* 执行时机： 必须等待网页中所有的内容加载完毕（包括图片）才能执行

* 编写个数： 不能同时编写多个

  ``` javascript
  window.onload = function(){
      alert('test1');
  }
  window.onload = function(){
      alert('test2');
  }
  结果只会输出"test2"
  ```

b.$(document).ready():

* 网页中所有DOM结构绘制完毕后就执行，可能DOM元素关联的东西并没有加载完

* 能同时编写多个

  ``` javascript
  $(document).ready(function(){
      alert('hello world!');
  });
  $(document).ready(function(){
      alert('hello again!');
  });
  结果两次都输出
  ```

* 简化写法：

  ``` javascript
  $(document).ready(function(){
      //...
  });
  可以简写成：
  $(function(){
      //....
  })
  //$(document)也可简写成$()。当$()不带参数时，默认参数是"document"
  $().ready(function(){
     //... 
  })

  ```




2.hover()方法

* 替代jQuery中的bind('mouseenter')和bind('mouseleave'),而不是替代bind('mouseover')和bind('mouseout')

* 当要触发hover()方法的第2个函数时，要用trigger('mouseleave')来触发，而不是trigger('mouseout')

  

3.return false作用

a.阻止事件对象冒泡

b.阻止事件对象的默认行为

c.相当于在事件对象上同时调用stopPrapagation()方法和preventDefault()方法的一种简写方式

> jQuery 不支持事件捕获

4. event.which

   * 在鼠标单击事件中获取到鼠标的左，中，右键； 

    ``` javascript
    $('a').mousedown(function(e){
    alert(e.which) // 1 = 鼠标左键  2 = 鼠标中键  3 = 鼠标右键
    })
    ```

   * 在键盘事件中获取键盘的按键

    ``` javascript
    $('input').keyup(function(e){
        alert(e.which);
    })
    ```


5.添加事件命名空间

``` javascript
$(function(){
    $('div').bind('click.plugin',function(){
        $('body').append('<p>click事件</p>')
    });
    $('div').bind('mouseover',function(){
        $('body').append("<p>mouseover事件</p>")
    });
    $('button').click(function({
        $('div').unbind('.plugin');
    }))
    
})
```

6.attr() 和prop()

* 只添加属性名称，该属性就会生效应该使用prop();
* 只存在true/false的属性应该使用prop();
* 设置disabled和checked这些属性，应使用prop();

###五、jQuery 中的ajax

1.load()方法： 能载入远程HTML代码并插入DOM中，结构为：

``` html
load(url [,data] [,callback])
```

* 传递方式： 根据data来自动指定，没有参数传递，采用GET方式传递，反之，自动转换为POST方式。

* 回调参数：3个，分别代表请求返回的内容，请求状态，XMLHttpRequest对象。

  ``` javascript
  $('body').load('test.html', function(responseText, textStatus, XMLHttpRequest){
      responseText: 请求返回的内容
      textStatus: 请求状态： success, error, notmodified, timeout 
      XMLHttpRequest: XMLHttpRequest对象
  })
  ```

2. $.get()：使用GET方式进行异步请求，结构：

   ``` javascript
   $.get(url [, data] [, callback] [, type])
   ```

   a.  url 为必选参数，其他为可选参数，

   b. 回调函数只有2个参数，data 和 textStatus 。只有当数据成功返回 (success) 后才被调用，这点与load()不一样

   c.  type指服务器返回内容的格式，包括xml, html, script, json, text 和_default

   

3. GET 和 POST区别

   * GET：将参数跟在URL后进行传递， POST：作为HTTP消息的实体内容发送给Web服务器
   * GET：对传输的数据大小有限制（<=2kb）,POST: 传递的数据量大
   * GET：请求数据会被浏览器缓存，其他人可以从历史记录中读取，会有严重的安全性问题

4. $.getScript(): 直接加载.js文件

   ``` javascript
   $(function(){
       $('#send').click(function(){
           $.getScript('test.js')
       })
   })
   ```

5. $.getJSON(): 用于加载JSON文件

6. 序列化元素：

   * serialize()
   * serializeArray()：将DOM元素序列化后，返回JSON格式的数据
   * $.param(): 对一个数组或对象按照key/value 进行序列化

### 六、jquery 插件 — Cookie

1. 写入Cookie： 

   ``` javascript
   $.cookie('the_cookie','the_value')
   ```

2. 读取Cookie

   ``` javascript
   $.cookie('the_cookie')
   ```

3. 删除Cookie

   ``` javascript
   $.cookie('the_cookie', null)
   ```

### 七、控制台面板

1.记录日志

console.log：简单记录日志

console.debug: 记录调试信息，并附上行号的超链接

console.error: 在消息前显示错误图标，附上行号的超链接

console.info: 在消息前显示信息图标，附上行号的超链接

console.warn: 在消息前显示警告图标，附上行号的超链接

2. 格式化字符串输出和多变量输出
   * %s :        字符串
   * %d, %i:   数字
   * %f:           浮点数
   * %o:            链接对象

### 八、 Ajax 的 XMLHttpRequest 对象的属性和方法

1. readyState属性： 一个XMLHttpRequest对象 被创建，readyState标识当前对象正处于的状态，具体值如下：

   * 0： 未初始化状态，已经创建XMLHttpRequest 对象，未被初始化

   * 1：准备发送状态，已经调用XMLHttpRequest对象的open()方法，准备将         请求发送到服务器

   * 2：已发送状态，已通过send()方法把请求发送到服务器，未收到响应

   * 3：正在接收状态，已接收到HTTP响应头信息，但消息体部分没有完全接收到

   * 4：完全响应状态， 已经完成了HttpResponse响应的接收

     

2. responseText属性：包含客户端接收到的HTTP响应的文本内容

   * readyState值为 0,1和2 ，responseText包含一个空字符串

   * readyState值为3， responseText中包含客户端未完成的响应信息

   * readyState值为4，responseText中包含完整的响应信息

     

3. responseXML属性： 描述被XMLHttpRequest解析的XML文档的属性

   > readyState值为4，响应头的Content-Type 的MIME被指定为XML，该属性才有值，并解析一个XML文档，否则为null

4. status 属性：描述HTTP状态代码（readyState值为3或4时才能访问）

5. statusText 属性：描述HTTP状态代码文本（readyState值为3或4时才能访问）

6. onreadystatechange 事件：触发回传处理函数（readyState值发生改变）

7. open()方法：初始化，得到一个可以用来进行发送的对象，参数如下

   * method: 必选参数，大写，指定发送请求的HTTP方法（GET，POST等）

   * uri：指定XMLHttpRequest发送请求的服务器地址，自动解析为绝对地址
   * async: 指定请求是否为异步，默认值为true
   * username和password：需要服务器验证访问用户情况时传

8. send()方法：1个可选参数，包含可变类型数据，使用setRequestHeader()先设置Content-Type头部，或显式使用null参数调用

9. abort()方法：暂停HttpRequest 请求发送或HttpResponse接收，将XMLHttpRequest对象设置为初始化

10. setRequestHeader()方法：设置请求头信息，包含两个参数，header键名，键值

    > 在readyState为1，调用open()方法后调用

11. getResponseHeader()方法： 检索响应头信息(在readyState为3或4)，

    或通过getAllResponseHeaders()获取所有HttpResponse头部信息