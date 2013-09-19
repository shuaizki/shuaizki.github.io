#html的二次学习

##样式
三种定义方法：引用，内部定义和内联  

    <style> 定义样式定义。  
    <link>  定义资源引用。
    <div>   定义文档中的节或区域（块级）。
    <span>  定义文档中的行内的小块或区域。
    <font>  规定文本的字体、字体尺寸、字体颜色。不赞成使用。请使用样式。
    <basefont>      定义基准字体。不赞成使用。请使用样式。
    <center>        对文本进行水平居中。不赞成使用。请使用样式。

##超链接a中的target
target给出一个标记，如果打开a的时候无此标记，那么新建一个窗口A并赋予其标记, 下次打开相同标记的超链接的时候，直接就会在窗口A内打开了。  

使用target更通常的方法是在一个frameset中将超链接内容重定向到一个或者多个框架中。  
显示文档的框架如下:  

    <frameset cols="100,*">
        <frame src="toc.html">
        <frame src="pref.html" name="view_frame">
    </frameset> 

其中toc.html的内容是  

    <h3>Table of Contents</h3>
    <ul>
      <li><a href="pref.html" target="view_frame">Preface</a></li>
      <li><a href="chap1.html" target="view_frame">Chapter 1</a></li>
      <li><a href="chap2.html" target="view_frame">Chapter 2</a></li>
      <li><a href="chap3.html" target="view_frame">Chapter 3</a></li>
    </ul>

保留目标:  
_self把目标文档载入并显示在相同的框架或者窗口中。  
_parent把目标文档载入父窗口或者包含该超链接引用的框架的框架集。如果引用在窗口或者顶级框架中，与楼上的东西等价。  
_top使得文档载入包含这个超链接的窗口。  

有用的提示：  
注释：请始终将正斜杠添加到子文件夹。假如这样书写链接：href="http://www.w3school.com.cn/html"，就会向服务器产生两次 HTTP 请求。这是因为服务器会添加正斜杠到这个地址，然后创建一个新的请求，就像这样：href="http://www.w3school.com.cn/html/"。  

##有关img
###align属性负责行内对齐  
<p>贴一张破解大神Stefan<img src="http://ww2.sinaimg.cn/bmiddle/612edf3ajw1e89g8dlnp6j20az0cyjrp.jpg" width="50" height="50" align="top"/> Esser的照片来演示行内对其</p>
<p>无节操的图片<img src="http://ww1.sinaimg.cn/bmiddle/612edf3ajw1e89goow42hj20by07yt9a.jpg" width="50" height="50" align="bottom"/>bottom对齐</p>
<p>无节操的图片<img src="http://ww1.sinaimg.cn/bmiddle/612edf3ajw1e89goow42hj20by07yt9a.jpg" width="50" height="50" align="left"/>来演示图像浮动</p>

#昨天已然逝去，今天继续开始09_04 

##表格
<table border="1">
<tr>
<th> heading r1,d1</th>
<th> heading r1,d2</th>
</tr>
<tr>
<td> row 1, data 1</td>
<td> row 1, data 2</td>
</tr>
<tr>
<td> row 2, data 1</td>
<td> row 2, data 2</td>
</tr>
</table>

空的单元格用空格占位符  
    
    <td>&nbsp;</td>

跨行或者跨列的的单元格用属性rowspan或者colspan  
跨列:  
<table>
<tr>
<th> 姓名</th> <th colspan="2">兵团名称</th>
</tr>
<tr>
<td>艾尔敏</td> <td>调查</td> <td>兵团</td>
</tr>
</table>

跨行的用法大同小异  

##html块
html div 没有特殊含义，就是块级元素，可用于对大的内容块进行样式布置。  
另一个常见的用途是文档布局，取代了使用表格定义布局的老式方法。  
span元素是内联元素，可用作文本容器，也没有特殊定义，设置样式blah..blah..  

##html布局
默认是换行，用float可以改变这一局面  

##表单
   
    <form>
    <input>各种乱七八糟的输入</input>
    </form>

##html框架
不能将frameset与body一起使用  
iframe是内联框架，也就是说可以用在body里  

##html base元素
为页面上所有连接规定默认地址或默认目标  

##html link元素
定义文档与外部资源之间的关系  

#html meta
只能存在于header中，记录页面的描述、关键词、文档的作者等等  

#html url
url只能使用ascii字符集  

#今天(09_05)接着搞
放视频放音频的就略过了  

##xhtml是以xml格式编写的html
是更加严格的html  
文档结构

        XHTML DOCTYPE 是强制性的
        <html> 中的 XML namespace 属性是强制性的
        <html>、<head>、<title> 以及 <body> 也是强制性的
        元素语法
        XHTML 元素必须正确嵌套
        XHTML 元素必须始终关闭
        XHTML 元素必须小写
        XHTML 文档必须有一个根元素
        属性语法
        XHTML 属性必须使用小写
        XHTML 属性值必须用引号包围
        XHTML 属性最小化也是禁止的

