	
	
	
	
	
	Network Working Group                                           C. Rigney
	Request for Comments: 2869                                     Livingston
	Category: Informational                                        W. Willats
	                                                        Cyno Technologies
	翻译：twingao                                                   P. Calhoun
	                                                         Sun Microsystems
	                                                                June 2000
	
	
	                              RADIUS扩展
	
	备忘录状态
	
	   本文不是在制定一个Internet标准，只是向互联网社区提供相关信息，本文可
	   以不受限制地传播。
	
	
	版权说明
	
	   Copyright (C) The Internet Society (2000).  All Rights Reserved.
	
	摘要
	
	   本文描述了利用RADIUS协议在NAS和共享的计费服务器之间传送认证、授权和计
	   费信息的额外属性。RADIUS协议在RFC 2865 [1]和RFC 2866 [2]中已经描述。
	
	 
	
	
	目录
	
	   1.     简介 ..................................................    2
	      1.1       描述文档的约定 ..................................    3
	      1.2       术语 ............................................    3
	   2.     操作 ..................................................    4
	      2.1       RADIUS对计费更新的支持 ..........................    4
	      2.2       RADIUS对Apple远程接入协议的支持 .................    5
	      2.3       RADIUS对扩展认证协议（EAP）的支持 ...............   11
	         2.3.1  协议概述 ........................................   11
	         2.3.2  重传 ............................................   13
	         2.3.3  分片 ............................................   14
	         2.3.4  举例 ............................................   14
	         2.3.5  Alternative uses ................................   19
	   3.     报文格式 ..............................................   19
	   4.     报文类型 ..............................................   19
	   5.     属性 ..................................................   20
	
	 
	
	 
	
	Rigney, et al.               Informational                      [Page 1]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      5.1       Acct-Input-Gigawords ............................   22
	      5.2       Acct-Output-Gigawords ...........................   23
	      5.3       Event-Timestamp .................................   23
	      5.4       ARAP-Password ...................................   24
	      5.5       ARAP-Features ...................................   25
	      5.6       ARAP-Zone-Access ................................   26
	      5.7       ARAP-Security ...................................   27
	      5.8       ARAP-Security-Data ..............................   28
	      5.9       Password-Retry ..................................   28
	      5.10      Prompt ..........................................   29
	      5.11      Connect-Info ....................................   30
	      5.12      Configuration-Token .............................   31
	      5.13      EAP-Message .....................................   32
	      5.14      Message-Authenticator ...........................   33
	      5.15      ARAP-Challenge-Response .........................   35
	      5.16      Acct-Interim-Interval ...........................   36
	      5.17      NAS-Port-Id .....................................   37
	      5.18      Framed-Pool .....................................   37
	      5.19      属性列表 .........................................  38
	   6.     IANA事项 ..............................................   39
	   7.     安全事项 ..............................................   39
	      7.1       Message-Authenticator安全 .......................   39
	      7.2       EAP安全 .........................................   39
	         7.2.1  Separation of EAP server and PPP authenticator ..   40
	         7.2.2  Connection hijacking ............................   41
	         7.2.3  中间人攻击 ......................................   41
	         7.2.4  多数据库 ........................................   41
	         7.2.5  协商攻击 ........................................   42
	   8.     参考文献 ..............................................   43
	   9.     致谢 ..................................................   44
	   10.    AAA工作组主席地址 .....................................   44
	   11.    作者地址 ..............................................   45
	   12.    版权声明 ..............................................   47
	
	1.  简介
	
	   RFC 2865 [1]描述了RADIUS协议，而且目前已经被实现和部署，RFC 2866 [2]
	   描述了如何使用RADIUS协议进行计费。
	
	
	  
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                      [Page 2]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   本备忘录建议了几个额外的属性，将它们增加到RADIUS协议中，可以实现多种
	   非常有用的功能。这几个属性目前还没有大面积使用的经验，因此只能看作是
	   实验性的。
	
	   扩展认证协议（EAP）[3]是对PPP协议的扩展，通过EAP可以在PPP协议内支持另
	   外的认证方法。本备忘录描述了RADIUS协议如何利用EAP-Message属性和
	   Message-Authenticator属性支持EAP。
	
	   所有的属性由不同长度的Type-Length-Value三元组组成。新的属性值的加入不
	   会影响到原有协议的实现。
	
	 
	
	 
	
	1.1.  描述文档的约定
	
	   本文中的关键词"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
	   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", 以及"OPTIONAL"的描述请
	   参见RFC 2119 [4]。
	
	   如果一种实现没有满足本协议中的一个或者多个MUST或者MUST NOT要求，那么
	   该实现和本协议是不兼容的；如果一个实现满足了本协议中所有的MUST、
	   MUST NOT、SHOULD和SHOULD NOT要求，那么该实现和本协议是无条件兼容；如
	   果该实现满足本协议所有的MUST和MUST NOT要求，但没有满足所有的SHOULD和
	   SHOULD NOT要求，那么该实现和本协议是有条件兼容。
	
	   不支持某个特定服务的NAS一定不要（MUST NOT）支持该服务的RADIUS属性，例
	   如：不能提供ARAP服务的NAS一定不要（MUST NOT）支持ARAP服务的RADIUS属
	   性，对于授权不支持的服务的接入成功回应报文，NAS必须将之做为接入拒绝回
	   应报文一样处理。
	
	 
	
	
	1.2.  术语
	
	   本文使用了以下的术语：
	
	   服务      NAS为拨入用户提供的某种服务，如：PPP或者Telnet。
	  
	
	   会话      NAS为拨入用户提供的每一个服务都会建立一个会话。第一次开始提
	             供服务做为会话的开始，服务终止做为会话的结束。如果NAS支持的
	             话，一个用户可以有多个并行或者串行的会话。
	
	 
	
	 
	
	
	Rigney, et al.               Informational                      [Page 3]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	 
	
	 
	
	
	   静默丢弃
	
	             应用程序不对包进行任何处理就直接丢弃。应用程序应该
	             （SHOULD）有提供记录错误的能力，其中包括被静默丢弃的包的内
	             容，而且应用程序应该（SHOULD）在一个统计计数器中记录下该事
	             件。
	
	2.  操作
	
	   操作同RFC 2865 [1]和RFC 2866 [2]中定义的完全相同。
	
	
	2.1.  RADIUS对计费更新的支持
	
	   当用户来认证时，RADIUS服务器回应一个Access-Accept报文表示接入请求成
	   功。如果服务器希望接到该用户的计费更新报文，那么在Access-Accept报文中
	   必须包含Acct-Interim-Interval属性。该属性指明了计费更新报文的时间间隔
	   （以秒为单位）。
	
	   当然，也可以在NAS上静态配置一个计费更新报文时间间隔。注意，NAS上的本
	   地配置必须（MUST）覆盖Access-Accept报文中携带的属性值。
	
	   这个方案并不会破坏后向兼容性。因为如果RADIUS服务器不支持该扩展就不会
	   添加该属性。同样，如果NAS不支持该扩展就会忽略该属性。
	
	   注意：在计费更新报文中所有信息是累积的，即：数据包的数量是从会话开始
	   到当前的总的数据包数量，而不是从上一个计费更新报文以后的数据包数量。
	
	   可以预见，计费更新报文中除了不包含Acct-Term-Cause属性外，包含了计费结
	   束报文中其它的所有属性。
	
	   既然所有的信息是累积的，NAS必须（MUST）确保在给定的任何时间在重发队列
	   中对于某一个会话只有一个单独的计费更新报文存在。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                      [Page 4]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   NAS可以（MAY）利用fudge因子在不同会话的计费更新报文之间增加一段随机时
	   延。这样避免了所有的报文在循环时间一到就立刻被发送。例如：如果一个主
	   要的链路最近刚恢复，很多拨号用户（原来使用该链路的拨号用户，会在同一
	   时刻重新拨号）立刻被指向同一个NAS。
	
	   应该（should）认证考虑使用计费更新（扩展特性）对网络和NAS CPU的造成的
	   负担，因此应该合理选择Acct-Interim-Interval属性的值。
	
	 
	
	2.2.  RADIUS对Apple远程接入协议的支持
	
	   RADIUS协议提供了一个允许多个NAS共享一个通用认证数据库的方法。
	
	   Apple远程接入协议（ARAP）提供一种在点到点链路上传送AppleTalk网络流量
	   的方法，点到点链路通常（但不是唯一）是异步和ISDN交换电路连接。尽管
	   Apple在未来的远程接入业务中朝着ATCP on PPP的方向发展，但对已经使用远
	   程接入的Macintosh用户来说，RARP仍然是一种通用的方法，而且可能要存在一
	   段时间。
	
	   有几个NAS供应商支持ARAP，同时，这些供应商在同一个NAS上也支持PPP、IPX
	   和其它协议。在这些支持多协议的设备上，ARAP连接通常不使用RADIUS协议做
	   为认证协议，即便使用RADIUS协议认证，每个供应商有各自的解决方案。
	
	   本节描述支持RARP协议的RADIUS几个额外属性。实现本规范的RADIUS客户端和
	   服务器端的实现应该在互操作的方式能够认证ARAPR连接。
	
	   本节的讨论假定读者对RADIUS协议已很熟悉，因此首先直接探究ARAP应用的
	   具体细节，然后再详细讨论RADIUS协议的ARAP额外属性。
	
	   本文档不讨论的两个ARAP特色如下：
	
	      1. 用户发起的密码改变。这不是RADIUS的一部分，除了RADIUS协议也可以
	         通过软件过程实现。
	
	      2. Out-of-Band消息。NAS任何时候都可以向ARA客户端发送消息，消息以对
	         话框的形式显示在拨号接入用户的屏幕上。这不属于认证的一部分，也
	         不属于此处讨论的范畴。但是我们注意到Access-Accept报文中的
	         Reply-Message属性可以下发做为登陆消息。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                      [Page 5]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	 
	
	 
	
	   我们尽可能多地尊重已有的RADIUS协议精神，使设计与以前的技术兼容。更进一步
	   讨论，我们需要在两方面作出平衡选择处理。一方面，过多的新属性造成RADIUS世
	   界的泛滥；另一方面，将整个ARAP操作隐藏在一个单独的复合ARAP属性串中，或者
	   在扩展认证协议（EAP）的机制内解决。
	
	   但是，我们认为只要保证几个类似命名的属性，ARAP从PPP分离出来就足够了。
	
	   我们已经假定能够理解ARAP的RADIUS服务器可以进行DES加密且可以产生安全模
	   式挑战字。这正与RADIUS的如下目标一致：灵活服务器/简单NAS。
	
	   ARAP对一个连接的认证分两个阶段。第一个阶段是利用用户密码作为密钥，互
	   换一个“二次DES”随机数。之所以说是“二次”，是因为ARAP NAS挑战拨号接入
	   用户对自己进行验证，同样，拨号接入用户挑战ARAP NAS从对自己进行验证。
	
	   特别地，ARAP执行如下过程：
	
	      1. NAS在ARAP的msg_auth_challenge分组中向拨号接入用户发送两个32位的
	         随机数。
	
	      2. 拨号接入用户收到NAS发来的两个随机数后，利用用户密码对这两个随机
	         数进行DES加 密，然后拨号接入客户端将此加密后的结果连同用户名和
	         客户端产生的两个32位的随机数 在ARAP msg_auth_request分组中回应
	         给NAS。
	
	      3. NAS接收到以后，确认由拨号接入客户端发送过来的经过加密的随机数是
	         否是自己所期望 的。如果是，它利用密码对拨号接入客户端发送过来的
	         挑战字进行加密，并且把加密后的 结果在ARAP msg_auth_response分组
	         中发给拨号接入客户端。
	
	 
	
	 
	
	 
	
	
	   注意如果拨号接入客户端的响应错误（这意味着用户密码错误），服务器可以
	   重发，直到重发次数达到NAS允许的最大发送次数。在这种情况下，当拨号接入
	   客户端接收到ARAP msg_auth_response分组后，将用ARAP msg_auth_again分组
	   进行确认。
	
	   第一个“DES Phase”阶段通过以后，ARAP NAS可以利用被Apple称之为
	   “Add-In Security Modules”的机制发起第二个认证阶段。Security Modules
	   是运行在客户端和服务器上的几小段代码，允许通过通讯链路读和写任意数
	   据，从而实现额外的认证功能。各种安全令牌提供商利用这种机制对ARA呼叫者
	   进行认证。
	
	
	Rigney, et al.               Informational                      [Page 6]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   尽管ARAP允许安全模块读写它们所需要的任意信息，但是已经存在的安全模块
	   是利用简单的挑战和响应循环，当然可能携带某些全局控制信息。本文档假定
	   利用一个或几个challenge/response循环可以支持所有已经存在的安全模块。
	
	   在DES Phase之后，Security Module phase之前，RAP向下发送某些概貌信息，
	   使RADIUS和ARAP集成复杂。这意味着，在某些异常时间，除了对challenge的响
	   应以外，还必须存在概貌信息。幸运的是这些信息只是几个与密码相关的数
	   字，本文档中把这些信息封装在一个单独的新的属性中。
	
	   向RADIUS发送一个接入请求报文代表ARAP连接是非常直接的。ARAP NAS产生一
	   个随机的数字挑战字，然后接受拨号接入客户端的响应，客户端的挑战字和用
	   户名。假定用户不是一个guest，在Access-Request分组中转发如下信息：
	   User-Name（最长31个字符），Framed-Protocol （对于ARAP，此域值填为
	   3），ARAP-Password和其它希望携带的属性，如Service-Type、
	   NAS-IP-Address、NAS-Id、NAS-Port-Type、NAS-Port、NAS-Port-Id和
	   Connect-Info等。
	
	   请求认证字是一个NAS产生的16字节的随机数。这个随机数的低8个字节作为两
	   个四字节的随机数放在ARAP msg_auth_challenge报文中传给拨号接入用户。其
	   中字节0-3作为第一个随机数，字节4-7作为第二个随机数。
	
	   接入请求报文中的ARAP-Password包含一个16 字节的随机数域，用来携带拨号
	   进入用户对NAS挑战的响应及客户端自己 对NAS的挑战字。高字节包含拨号接入
	   用户对NAS的挑战字（2个32位的数字，共8字节），第字节包含拨号接入用户对
	   NAS挑战的响应（2个32位的数字，共8字节）。
	
	   User-Password，CHAP-Password或ARAP-Password三个属性在接入请求报文中只
	   能出现一个，也可以出现一个或多个EAP-Messages。
	
	   如果RADIUS服务器不支持ARAP，它应该向NAS返回一个接入拒绝回应报文。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                      [Page 7]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   如果RADIUS服务器不支持ARAP，它应该利用Challenge（请求认证字中的低8个
	   字节）和用户响应（ARAP-Password中的低8个字节）来确认用户的响应。
	
	   如果认证失败，RADIUS服务器应该向NAS返回一个接入拒绝回应报文，报文中携
	   带可选属性Password-Retry和Reply-Messages。如果携带Password-Retry属
	   性，这就告知ARAP NAS可以选择再发起几个挑战-响应循环，直到循环次数等于
	   Password-Retry属性中的整数值。
	
	   如果用户认证通过，RADIUS服务器应该向NAS返回一个Access-Accept报文
	   （Code等于2），ID和Response Authenticator跟通常情况一样，其它属性如
	   下：
	
	      Service-Type of Framed-Protocol
	
	      Framed-Protocol of ARAP (值为3)
	
	      Session-Timeout，它代表用户可以连接的最长时间（以秒为单位）。如果
	      用户被授权为无限 制用户，那么在Access-Accept报文中就不应该包含
	      Session-Timeout属性。此时ARAP将用户作为无限制用户超时（值为-1）。
	
	
	      ARAP-Challenge-Response，它包含8个字节，代表对拨号接入用户的响应。
	      RADIUS是用如下方法填充出此属性的。RADIUS服务器从ARAP-Password属性
	      中取出高8个字节（实际为拨号接入用户的挑战），然后再用认证用户的密
	      码作为密钥，对前边已取出的用户的挑战进行DES加密。如果用户的密码在
	      长度上小于8个字节，那么就在用户密码填充NULL字节，直到满8个字节；如
	      果用户密码长度大于8个字节，那就应该返回一个Access-Reject报文。
	
	      ARAP-Features，它包含了NAS在ARAP“feature flags”报文中将携带给用
	      户的信息。
	
	         字节0：如果值为0，表示用户不能改变密码，如果值不为0，表示用户可
	         以改变密码（RADIUS不处理密码更改，仅用属性指明ARAP是否可以改变
	         密码）。
	
	         字节1：表示最小的可接受的密码长度（0-8）。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                      [Page 8]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	         字节2-5：表示Macintosh格式的密码产生日期，为一32位的无符号整
	         数，代表从Midnight GMT January 1, 1904以后的总的时间（以秒为单
	         位）。
	
	         字节6-9：表示从密码产生以后的有效时间长度（以秒为单位）。
	
	
	         字节10-13：以Macintosh格式表示的目前RADIUS时间。
	
	      作为可选项，回应报文中可以包含Reply-Message属性，其内容为一字符
	      串，最多可以包含253个字符，这些内容可以在对话框中显示给用户。
	
	      Framed-AppleTalk-Network：此属性可以包含在报文中。
	
	      Framed-AppleTalk-Zone：此属性可包含在报文中，最大长度为32个字符。
	
	      对于一个用户，ARAP定义了区域列表。与区域名字列表一起，ARAP定义了区
	      域访问标志（被NAS使用），此标志说明了如何利用区域名字列表。即：拨
	      号接入用户可能只允许访问缺省区域，或者只能访问区域列表中区域，也或
	      者只能访问除区域列表中列出的以外的其它区域。
	
	      ARAP NAS中含有一个指定的滤波器，用此滤波器可以处理此问题，其中滤波
	      器起码的区域名字。用此机制，解决了如下问题：用一个单独的RADIUS服务
	      器管理不同的NAS客户端，并且客户端有或许不能全部看到用户区域列表中
	      的区域名字。区域名字只对NAS才有意义。这种方法的不足之处在于滤器必
	      须在首先NAS中以某种方式启动，然后再由RADIUS Filter-Id引用。
	
	      ARAP-Zone-Access属性中包含一个整数，此属性指明了该如何使用针对一个
	      用户的区域列表该。如果包含此属性且其值为2或4，那么就必须包含
	      Filter-Id属性，Filter-Id属性命名了一个适用访问标记的滤波器。
	
	      在Access-Accept报文中包含Callback-Number或Callback-Id属性可能导致
	      ARAP NAS在发送Feature Flags开始回调处理后切断连接。其中回调处理是
	      以一种ARAP特有的方式进行。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                      [Page 9]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   在Access-Accept报文中也可以包含其它属性。
	
	   ARAP要完成到客户端拨号接入的连接，还需要其它信息。这种信息可由ARAP
	   NAS通过SNMP配置，NAS管理程序或从NAS的AppleTalk堆栈中获得，不需要
	   RADIUS的任何帮助。特别地：
	
	      1. 将AppearAsNet和AppearAsNode值传送给客户端，告诉它在数据报分组中
	         应该适用什么样的网络和节点值。 AppearAsNet可以从
	         Framed-AppleTalk-Network属性中获得，或者通过配置，或者通过NAS的
	         堆栈获得。
	
	      2. 拨号接入终端将出现在缺省区域内，缺省区域也就是AppleTalk的名字。
	      （或者用Framed-AppleTalk-Zone属性指明）。
	
	      3. 其它的NAS特有的资料，如NAS的名字和smartbuffering信息。
	      （Smartbuffering是ARAP的一种机制。它用小的令牌取代普通的AppleTalk
	      数据报，从而改善了某些普通慢链路的性能。）
	
	      4. 用户的“Zone List”信息。 ARAP规范定义了一个“zone count”域，
	      实际上没有使用。
	
	 
	
	 
	
	 
	
	   RADIUS用以下方式支持ARAP Security Modules。
	
	   DES认证结束以后，RADIUS服务器通知ARAP NAS为拨号接入用户运行一个或多个
	   security modules。尽管基本的协议支持连续执行多个security modules，但
	   在实践中，目前的实现只允许实现一个。通过使用多个Access-Challenge请
	   求，就可以支持多个安全模式，但这种功能可能永远不会被使用。
	
	   同时我们也假定，尽管ARAP允许在点到点链路的端点上安全模式之间使用自由
	   格式的对话，但在实践中，所有的安全模式可以简化为简单的挑战/响应循环。
	
	   如果RADIUS服务器希望通知ARAP NAS运行一个安全模式，那么它应该给NAS发送
	   一个Access-Challenge报文，作为可选项，报文中可以携带State属性，加上
	   ARAP-Challenge-Response ，ARAP-Features和其它两个属性：
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 10]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   ARAP-Security：一个四字节的模式签名，包含一个Macintosh OSType。
	
	   ARAP-Security-Data：一个字符串，携带了实际的模式挑战和响应。
	
	   当执行完安全模式，NAS向RADIUS再发送一个Access-Request报文，报文中的属
	   性ARAP-Security-Data包含了安全模式的响应，同时也包含Access-Challenge
	   中得到的State属性。 在这种情况下，authenticator域内不再包含特别的信
	   息，因为可以由存在的State属性来辨别。
	
	 
	
	 
	
	2.3.  RADIUS对扩展认证协议（EAP）的支持
	
	   扩展认证协议（EAP）描述在参考文献[3]中，它提供了在PPP协议内支持额外的
	   认证方式的一种标准机制。通过使用EAP，可以支持许多额外的认证方案，其中
	   包括智能卡（Smart card），Kerberos，Public Key，一次一密（One Time
	   Passwords）以及其它种类。为了在RADIUS协议内支持EAP，本文档引入了两个
	   新的属性：EAP-Message和Message-Authenticator属性。本节描述了RADIUS协
	   议如何利用这两个新的属性来支持EAP。
	
	   在提议的方案中，利用RADIUS服务器在NAS和后端安全服务器之间传送封装在
	   RADIUS内的EAP报文。尽管在RADIUS服务器和后端服务器之间的会话通常使用的
	   是后端服务器开发的私有协议，但通过将EAP报文封装在RASIUS报文的
	   EAP-Message属性内也是可能的。这样的优点是RADIUS服务器为了支持EAP，不
	   需要增加针对某种认证的代码，这些针对某种认证的代码的可以放在后端安全
	   服务器上。
	
	 
	
	 
	
	 
	
	2.3.1.  协议概述
	
	   认证对等端（拨号接入用户）和NAS之间的EAP会话以LCP中的EAP协商开始。一
	   旦EAP协商通过，NAS必须向认证对等端发送一个EAP-Request/Identity报文，
	   除非通过其它途径如Called-Station-Id或Calling-Station-Id确定了身份。认
	   证对等端然后向NAS发送一个EAP-Response/Identity报文，NAS收到此报文后将
	   之封装在RADIUS协议的Access-Request报文中的EAP-Message属性内转发给
	   RADIUS服务器。RADIUS服务器通常利用EAP-Response/Identity来断定将对用户
	   使用何种EAP类型。
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 11]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   为了允许不能理解EAP的RADIUS代理转发接入请求报文，如果NAS发送
	   EAP-Request/Identity，NAS必须（MUST）将EAP-Response/Identity中的内容
	   拷贝到User-Name属性中，同时在随后的每一个接入请求报文中的User-Name属
	   性中都必须（MUST）包含EAP-Response/Identity。NAS-Port和NAS-Port-Id属
	   性也应该（SHOULD）包含在由NAS发送的接入请求报文中，而NAS-Identifier或
	   NAS-IP-Address属性则必须（MUST）包括在内。为了允许不理解EAP的代理能够
	   转发接入回应报文，如果在接入请求报文中包含User-Name属性，则RADIUS服务
	   器在随后的接入成功回应报文中必须包含User-Name属性。如果没有User-Name
	   属性，计费和生成帐单将变得非常难于管理。
	
	   如果通过其它途径如Called-Station-Id或Calling-Station-Id确定了身份，
	   则NAS在每一个接入请求报文中必须（MUST）包含这些身份属性。
	
	   尽管这种方法将节省一个回环，但是不能普遍部署。在某些情况下，用户的身
	   份可能不（may not）需要确认（如当认证和计费基于Called-Station-Id或
	   Calling-Station-Id属性时），因此NAS就不必向认证对等端方发送
	   EAP-Request/Identity报文。当NAS不需要发送EAP-Request/Identity报文时，
	   NAS将向RADIUS服务器发送一个RADIUS接入请求报文，其中携带EAP-Message属
	   性，意味着EAP-Start。通过发送一个2字节的EAP-Message属性（没有数据）来
	   表明EAP-Start。然而需要注意的是：由于接入请求报文中没有携带User-Name
	   属性，这种方法同[1]中描述的RADIUS不兼容，同时也不能容易地应用在部署代
	   理的情况，如漫游和共享使用网络。
	
	   如果RADIUS服务器支持EAP，它必须（MUST）用携带了EAP-Message属性的接入
	   挑战报文响应，如果RADIUS服务器不支持EAP，它必须（MUST）用接入拒绝回应
	   报文响应。EAP-Message属性包含了一个封装的EAP报文，并传送给认证对等
	   端。如果NAS没有向认证对等端发起一个EAP-Request/Identity报文，那么接入
	   挑战报文通常会包含一个EAP-Message属性，此属性中封装了一个
	   EAP-Request/Identity消息，请求拨号接入终端确定自己的身份。此时NAS以包
	   含一个封装了EAP-Response的EAP-Message属性的接入请求报文作为响应，会话
	   一直持续到接收到一个RADIUS接入拒绝回应报文或接入成功回应报文。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 12]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   一旦收到一个RADIUS接入拒绝回应报文，不管携带还是没有携带一个封装了
	   EAP-Failure的EAP-Message属性，都必须（MUST）导致NAS向认证对等端发送一
	   个LCP中断请求报文。如果收到一个RADIUS接入成功回应报文，并且报文中携带
	   了一个封装有EAP-Success的EAP-Message属性，则认证阶段宣布结束。RADIUS
	   Access-Accept/EAP-Message/EAP-Success报文中必须包含所有期望的由接入成
	   功回应报文带回的属性。
	
	   上面所描述的场景中，NAS从来不需要操作EAP。另外一种替代方案是
	   EAP-Request/Identity报文总是由NAS向认证对等端发送。
	
	   对于代理的RADIUS请求，有两种处理方法。如果基于Called-Station-Id来确定
	   域，那么RADIUS服务器可以（may）代理最初的RADIUS
	   Access-Request/EAP-Start。如果域是基于用户身份来确定的，那么本地
	   RADIUS服务器必须（MUST）用一个RADIUS Access-Challenge/EAP-Identity报
	   文响应。从认证对等端返回的响应必须（MUST）被代理给最终的认证服务器。
	
	   对于代理的RADIUS请求，NAS可能收到一个接入拒绝回应报文，此报文是对
	   Access-Request/EAP-Identity报文的响应。如果请求报文被代理给一个不支持
	   EAP-Message扩展的RADIUS服务器，上述情况就会发生。一旦收到一个接入拒绝
	   回应报文，NAS就必须向认证对等端发送一个LCP Terminate Request
	   报文，中断连接。
	
	 
	
	 
	
	 
	
	
	2.3.2.  重传
	
	   如同[3]中所描述的那样，EAP认证者（NAS）负责认证对等端和NAS之间的报
	   文的重传。因此一旦一个报文从认证对等端传到NAS的过程中丢失（反之亦然），
	   NAS将重传。如同RADIUS [1]中所描述的，RADIUS客户端负责RADIUS客户端和
	   RADIUS服务器之间的报文重传。
	
	   需要注意的是，在某些情况下有必要调整重传策略和认证超时时间。例如，如果
	   使用令牌卡，那么就必须给用户留出额外的时间以便用户找到卡且输入卡的代
	   号。由于NAS通常不了解所需要的参数，因此这些参数需要RADIUS服务器提供。
	   这个问题可以这样解决：在接入挑战报文中包含Session-Timeout和
	   Password-Retry属性。
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 13]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   如果在接入挑战报文中包含了Session-Timeout属性，而且也包含了
	   EAP-Message属性，Session-Timeout的值指出了NAS在向拨号接入用户重传
	   EAP-Message报文之前，需要等待EAP-Response报文的最大秒数。
	
	 
	
	2.3.3.  分片
	
	   利用EAP-Message属性，RADIUS服务器可能将一个大于NAS和认证对等端之间链
	   路MTU的EAP报文封装在属性内。由于RADIUS服务器不可能利用MTU发现来确定链
	   路MTU，因此可以在一个携带EAP-Message属性的接入请求报文中再携带
	   Framed-MTU属性，由此可以给RADIUS服务器提供信息。
	
	 
	
	
	2.3.4.  举例
	
	   下面的例子给出了认证对等端、NAS和RADIUS服务器之间的会话。例中使用一次
	   一密（OTP）方式进行认证。使用用OTP认证方式，只是为了演示的目的，也可
	   以使用其它认证协议。若用其它认证协议，行为在某些方面可能稍微不同。
	
	 
	
	Authenticating Peer     NAS                    RADIUS Server
	-------------------     ---                    -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP ACK-EAP
	auth ->
	                        <- PPP EAP-Request/
	                        Identity
	PPP EAP-Response/
	Identity (MyID) ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/
	                        EAP-Response/
	                        (MyID) ->
	                                                <- RADIUS
	                                                Access-Challenge/
	                                                EAP-Message/EAP-Request
	                                                OTP/OTP Challenge
	                        <- PPP EAP-Request/
	                        OTP/OTP Challenge
	PPP EAP-Response/
	OTP, OTPpw ->
	
	 
	
	Rigney, et al.               Informational                     [Page 14]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/
	                        EAP-Response/
	                        OTP, OTPpw ->
	                                                 <- RADIUS
	                                                 Access-Accept/
	                                                 EAP-Message/EAP-Success
	                                                 (other attributes)
	                        <- PPP EAP-Success
	PPP Authentication
	Phase complete,
	NCP Phase starts
	
	如果NAS首先向RADIUS服务器发送一个EAP-Start报文给服务器，那么会话过程如
	下：
	
	Authenticating Peer     NAS                    RADIUS Server
	-------------------     ---                    -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP ACK-EAP
	auth ->
	                        RADIUS
	                        Access-Request/
	                       EAP-Message/Start ->
	                                               <- RADIUS
	                                               Access-Challenge/
	                                               EAP-Message/Identity
	                        <- PPP EA-Request/
	                        Identity
	PPP EAP-Response/
	Identity (MyID) ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/
	                        EAP-Response/
	                        (MyID) ->
	                                                <- RADIUS
	                                                Access-Challenge/
	                                                EAP-Message/EAP-Request
	                                                OTP/OTP Challenge
	                        <- PPP EAP-Request/
	                        OTP/OTP Challenge
	PPP EAP-Response/
	OTP, OTPpw ->
	
	 
	
	
	Rigney, et al.               Informational                     [Page 15]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/
	                        EAP-Response/
	                        OTP, OTPpw ->
	                                                 <- RADIUS
	                                                 Access-Accept/
	                                                 EAP-Message/EAP-Success
	                                                 (other attributes)
	                        <- PPP EAP-Success
	PPP Authentication
	Phase complete,
	NCP Phase starts
	
	如果客户端EAP认证失败，会话过程如下：
	
	
	Authenticating Peer     NAS                    RADIUS Server
	-------------------     ---                    -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP ACK-EAP
	auth ->
	                        Access-Request/
	                        EAP-Message/Start ->
	                                               <- RADIUS
	                                               Access-Challenge/
	                                               EAP-Message/Identity
	                        <- PPP EAP-Request/
	                        Identity
	PPP EAP-Response/
	Identity (MyID) ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/
	                        EAP-Response/
	                        (MyID) ->
	                                                <- RADIUS
	                                                Access-Challenge/
	                                                EAP-Message/EAP-Request
	                                                OTP/OTP Challenge
	                        <- PPP EAP-Request/
	                        OTP/OTP Challenge
	PPP EAP-Response/
	OTP, OTPpw ->
	                        RADIUS
	                        Access-Request/
	
	 
	
	Rigney, et al.               Informational                     [Page 16]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	                        EAP-Message/
	                        EAP-Response/
	                        OTP, OTPpw ->
	                                                 <- RADIUS
	                                                 Access-Reject/
	                                                 EAP-Message/EAP-Failure
	
	                        <- PPP EAP-Failure
	                        (client disconnected)
	
	如果RADIUS服务器或者代理不支持EAP-Message，会话过程如下：
	
	
	Authenticating Peer     NAS                       RADIUS Server
	-------------------     ---                       -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP ACK-EAP
	auth ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/Start ->
	                                                  <- RADIUS
	                                                  Access-Reject
	                        <- PPP LCP Terminate
	                        (User Disconnected)
	
	如果本地RADIUS服务器支持EAP-Message，而远端RADIUS服务器不支持，会话过程
	如下：
	
	
	Authenticating Peer     NAS                       RADIUS Server
	-------------------     ---                       -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP ACK-EAP
	auth ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/Start ->
	                                                  <- RADIUS
	                                                  Access-Challenge/
	                                                  EAP-Message/Identity
	                        <- PPP EAP-Request/
	                        Identity
	
	 
	
	
	Rigney, et al.               Informational                     [Page 17]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	PPP EAP-Response/
	Identity
	(MyID) ->
	                        RADIUS
	                        Access-Request/
	                        EAP-Message/EAP-Response/
	                        (MyID) ->
	                                                  <- RADIUS
	                                                  Access-Reject
	                                                  (proxied from remote
	                                                   RADIUS Server)
	                        <- PPP LCP Terminate
	                        (User Disconnected)
	
	如果认证对等端不支持EAP，但需要对用户用EAP认证，会话过程如下：
	
	 
	
	Authenticating Peer     NAS                       RADIUS Server
	-------------------     ---                       -------------
	
	                        <- PPP LCP Request-EAP
	                        auth
	PPP LCP NAK-EAP
	auth ->
	                        <- PPP LCP Request-CHAP
	                        auth
	PPP LCP ACK-CHAP
	auth ->
	                        <- PPP CHAP Challenge
	PPP CHAP Response ->
	                        RADIUS
	                        Access-Request/
	                        User-Name,
	                        CHAP-Password ->
	                                                  <- RADIUS
	                                                  Access-Reject
	                        <-  PPP LCP Terminate
	                        (User Disconnected)
	
	如果NAS不支持EAP，而需要对用户使用EAP认证，会话过程如下：
	
	
	Authenticating Peer     NAS                       RADIUS Server
	-------------------     ---                       -------------
	
	                        <- PPP LCP Request-CHAP
	                        auth
	
	 
	
	Rigney, et al.               Informational                     [Page 18]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	PP LCP ACK-CHAP
	auth ->
	                        <- PPP CHAP Challenge
	PPP CHAP Response ->
	                        RADIUS
	                        Access-Request/
	                        User-Name,
	                        CHAP-Password ->
	
	                                                 <- RADIUS
	                                                 Access-Reject
	                        <-  PPP LCP Terminate
	                        (User Disconnected)
	
	2.3.5.  选择使用
	
	   目前，由于缺乏标准化，后端安全服务器和RADIUS服务器之间的会话是私有
	   的。为了提高标准化程度，提供RADIUS供应商和后端安全提供商之间的互操
	   作，推荐（recommended）在会话中将EAP报文封装在RADIUS内。
	
	   这样作的好处是RADIUS服务器为了支持EAP，不需要有和某种特定认证有关的代
	   码，与特定认证有关的代码可以放在后端安全服务器上。
	
	   如果RADIUS服务器和后端安全服务器之间的会话使用RAIUS封装的EAP报文，那
	   么后端安全服务器通常会返回一个Access-Accept/EAP-Success报文，报文中不
	   需要包含当前接入成功回应报文中返回的属性。这意味着RADIUS服务器在向NAS发
	   送Access-Accept/EAP-Success报文之前，必须添加这些属性。
	
	 
	
	 
	
	 
	
	 
	
	3.  报文格式
	
	   报文格式同RFC 2865 [1]和RFC 2866 [2]中描述的完全相同。
	
	
	4.  报文类型
	
	   报文类型同RFC 2865 [1]和RFC 2866 [2]中描述的完全相同。
	
	   参见下面的“属性表”来确定哪种报文类型可以包含此处定义的哪个属性。
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 19]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	5.  属性
	
	   RADIUS属性通过请求报文和回应报文携带了关于的认证、授权和计费方面的信
	   息细节。
	
	   有些属性可以（MAY）被多次包含。这通常有特别的目的，这种情况都会在每个
	   属性的描述中特别指出。应该（SHOULD）保持相同类型的属性的顺序，不需要
	   保持不同类型的属性的顺序，
	
	   属性格式的概述和RFC 2865 [1]中描述的相同，但为了引用方便，此处也列出
	   了属性，各域是自左向右传输的。
	
	 
	
	 
	
	 
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |  Value ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	
	   类型
	
	      Type占位一个字节。到目前为止，在最新的“Assigned Number” RFC [5]
	      中给出了RADIUS Type的值的详细描述。值192-223是保留给实验用的，值
	      224-240是保留给特定实现用的，值241-255是预留的，而且不应该（should
	      not）使用它们。本文中涉及到以下的Type值：
	
	           1-39   (refer to RFC 2865 [1], "RADIUS")
	          40-51   (refer to RFC 2866 [2], "RADIUS Accounting")
	          52      Acct-Input-Gigawords
	          53      Acct-Output-Gigawords
	          54      Unused
	          55      Event-Timestamp
	          56-59   Unused
	          60-63   (refer to RFC 2865 [1], "RADIUS")
	          64-67   (refer to [6])
	          68      (refer to [7])
	          69      (refer to [6])
	          70      ARAP-Password
	          71      ARAP-Features
	          72      ARAP-Zone-Access
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 20]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	          73      ARAP-Security
	          74      ARAP-Security-Data
	          75      Password-Retry
	          76      Prompt
	          77      Connect-Info
	          78      Configuration-Token
	          79      EAP-Message
	          80      Message-Authenticator
	          81-83   (refer to [6])
	          84      ARAP-Challenge-Response
	          85      Acct-Interim-Interval
	          86      (refer to [7])
	          87      NAS-Port-Id
	          88      Framed-Pool
	          89      Unused
	          90-91   (refer to [6])
	          92-191  Unused
	
	   Length（长度）
	
	      Length域占位一个字节，表示包括Type、Length、Value域在内的属性的长
	      度。如果接收到的报文中的属性长度无效，整个请求报文必须（MUST）静默
	      丢弃。
	
	
	   Value（值）
	
	      Value域占位零个或者更多字节，包含属性信息的详细描述。值域的格式和
	      长度是由属性的类型和长度决定的。
	
	      需要指出的是，在RADIUS中没有任何类型的属性值是以NUL（十六进制的
	      0x00）结束的。譬如，“text”和“string"类型的属性值是不能以NUL结
	      束。由于属性具有长度域，因而不必使用结束符。“text”类型的属性值包
	      含UTF-8编码的10646 [8]字符，而“string”类型的属性值含有8位二进制
	      数据。服务器和客户端必须（MUST）能够处理嵌入的null。使用C语言实现
	      的RADIUS在处理字符串时需要注意不能使用strcpy()函数。
	
	      值域有五种数据类型。注意：类型“text”是类型“string"的一个子集。
	
	      text     1-253个字节，包含UTF-8编码的10646 [7]字符。长度为零的text
	               类型的属性不能（MUST NOT）被发送，而应该将整个属性忽略。
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 21]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      string   1-253个字节，包含二进制数据（值从0到255，十进制）。长度为
	               零的string类型的属性不能（MUST NOT）发送，而应该将整个属
	               性忽略。
	
	      address  32位的数值，最重要的字节优先传输。
	
	      integer  32位的无符号数，最重要的字节优先传输。
	
	      time     32位的无符号数，最重要的字节优先。从格林威治时间1970年1月
	               1日0时0分0秒时起的秒数。
	
	5.1.  Acct-Input-Gigawords
	
	   描述
	
	      这个属性描述了从提供服务以来，Acct-Input-Octets属性计数器由于超过
	      2^32而反转过多少次，该属性只能出现在Acct-Status-Type属性值为Stop
	      （计费结束报文）或Interim-Update（计费更新报文）的计费请求报文中。
	
	   Acct-Input-Gigawords属性的格式如下所示。各个域是按照自左向右的顺序传
	   输的。
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      52 代表Acct-Input-Gigawords.
	
	   长度
	
	      6
	
	   值
	
	      值域占位4个字节。
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 22]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	5.2.  Acct-Output-Gigawords
	
	   描述
	
	      这个属性描述了从提供服务以来，Acct-Output-Octets属性计数器由于超过
	      2^32而反转过多少次，该属性只能出现在Acct-Status-Type属性值为Stop
	      （计费结束报文）或Interim-Update（计费更新报文）的计费请求报文中。
	
	   Acct-Output-Gigawords属性的格式如下所示。各个域是按照自左向右的顺序传
	   输的。
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      53 代表Acct-Output-Gigawords.
	
	   长度
	
	      6
	
	   值
	
	      值域占位4个字节。
	
	
	5.3.  Event-Timestamp
	
	   描述
	
	      这个属性包含在Accounting-Request报文中，记录了事件在NAS上的发生时
	      间，从1970年1月1日0时0分0秒开始的秒数。
	
	
	   Event-Timestamp属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 23]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      55 代表Event-Timestamp
	
	   长度
	
	      6
	
	   值
	
	      值域占位4个字节的无符号整数，为从1970年1月1日0时0分0秒开始的秒数。
	
	
	5.4.  ARAP-Password
	
	   Description
	
	      This attribute is only present in an Access-Request packet
	      containing a Framed-Protocol of ARAP.
	
	      Only one of User-Password, CHAP-Password, or ARAP-Password needs
	      to be present in an Access-Request, or one or more EAP-Messages.
	
	   A summary of the ARAP-Password attribute format is shown below.  The
	   fields are transmitted from left to right.
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	                                   |             Value2
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	                                   |             Value3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	                                   |             Value4
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	                                   |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	
	Rigney, et al.               Informational                     [Page 24]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Type
	
	      70 for ARAP-Password.
	
	   Length
	
	      18
	
	   Value
	
	      This attribute contains a 16 octet string, used to carry the
	      dial-in user's response to the NAS challenge and the client's own
	      challenge to the NAS.  The high-order octets (Value1 and Value2)
	      contain the dial-in user's challenge to the NAS (2 32-bit numbers,
	      8 octets) and the low-order octets (Value3 and Value4) contain the
	      dial-in user's response to the NAS challenge (2 32-bit numbers, 8
	      octets).
	
	5.5.  ARAP-Features
	
	   Description
	
	      This attribute is sent in an Access-Accept packet with Framed-
	      Protocol of ARAP, and includes password information that the NAS
	      should sent to the user in an ARAP "feature flags" packet.
	
	   A summary of the ARAP-Features attribute format is shown below.  The
	   fields are transmitted from left to right.
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     Value1    |    Value2     |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                           Value3                              |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                           Value4                              |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                           Value5                              |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   Type
	
	      71 for ARAP-Features.
	
	   Length
	
	      16
	
	 
	
	Rigney, et al.               Informational                     [Page 25]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Value
	
	      The Value field is a compound string containing information the
	      NAS should send to the user in the ARAP "feature flags" packet.
	
	         Value1: If zero, user cannot change their password. If non-zero
	         user can.  (RADIUS does not handle the password changing, just
	         the attribute which indicates whether ARAP indicates they can.)
	
	         Value2: Minimum acceptable password length, from 0 to 8.
	
	         Value3: Password creation date in Macintosh format, defined as
	         32 unsigned bits representing seconds since Midnight GMT
	         January 1, 1904.
	
	         Value4: Password Expiration Delta from create date in seconds.
	
	         Value5: Current RADIUS time in Macintosh format.
	
	5.6.  ARAP-Zone-Access
	
	   Description
	
	      This attribute is included in an Access-Accept packet with
	      Framed-Protocol of ARAP to indicate how the ARAP zone list for the
	      user should be used.
	
	   A summary of the ARAP-Zone-Access attribute format is shown below.
	   The fields are transmitted from left to right.
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	
	   Type
	
	      72 for ARAP-Zone-Access.
	
	   Length
	
	      6
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 26]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Value
	
	      The Value field is four octets encoding an integer with one of the
	      following values:
	
	      1      Only allow access to default zone
	      2      Use zone filter inclusively
	      4      Use zone filter exclusively
	
	
	      The value 3 is skipped, not because these are bit flags, but
	      because 3 in some ARAP implementations means "all zones" which is
	      the same as not specifying a list at all under RADIUS.
	
	      If this attribute is present and the value is 2 or 4 then a
	      Filter-Id must also be present to name a zone list filter to apply
	      the access flag to.
	
	5.7.  ARAP-Security
	
	   Description
	
	      This attribute identifies the ARAP Security Module to be used in
	      an Access-Challenge packet.
	
	   A summary of the ARAP-Security attribute format is shown below.  The
	   fields are transmitted from left to right.
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   Type
	
	      73 for ARAP-Security.
	
	   Length
	
	      6
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 27]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Value
	
	      The Value field is four octets, containing an integer specifying
	      the security module signature, which is a Macintosh OSType.
	      (Macintosh OSTypes are 4 ascii characters cast as a 32-bit
	      integer)
	
	5.8.  ARAP-Security-Data
	
	   Description
	
	      This attribute contains the actual security module challenge or
	      response, and can be found in Access-Challenge and Access-Request
	      packets.
	
	   A summary of the ARAP-Security-Data attribute format is shown below.
	   The fields are transmitted from left to right.
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     String...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   Type
	
	      74 for ARAP-Security-Data.
	
	   Length
	
	      >=3
	
	   String
	
	      The String field contains the security module challenge or
	      response associated with the ARAP Security Module specified in
	      ARAP-Security.
	
	5.9.  Password-Retry
	
	   描述
	
	      这个属性可以（MAY）出现在接入拒绝回应报文中，用来指明用户在被中断以前可
	      以被允许尝试多少次认证。
	      
	      此属性最初主要是给ARAP认证使用的。
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 28]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Password-Retry属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      75 代表Password-Retry.
	
	   长度
	
	      6
	
	   值
	
	      值域占位4个字节，包含了允许用户重试密码的次数
	
	
	5.10.  Prompt
	
	   描述
	
	      这个属性只能用在接入挑战报文中，它向NAS指明当用户输入时，NAS是否应
	      该回显用户的响应。
	
	 
	
	   Prompt属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      76 代表Prompt.
	
	 
	
	
	Rigney, et al.               Informational                     [Page 29]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   长度
	
	      6
	
	   值
	
	      值域占位4个字节
	
	       0      No Echo
	       1      Echo
	
	5.11.  Connect-Info
	
	   描述
	
	      这个属性由NAS发送，指明了用户连接的特性。
	
	      NAS可以在接入请求报文或计费请求报文中包含此属性，指明用户连接的特
	      性。
	
	 
	
	   Connect-Info属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     Text...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      77 代表Connect-Info.
	
	   长度
	
	      >= 3
	
	   文本
	
	      文本域包含UTF-8编码的10646 [8]字符。报文中第一个Connect-Info属性的
	      开始部分应该（SHOULD）包括连接速度。如果发送和接收连接速度不同，那
	      么它们可以全部包含在第一个属性内，首先为发送速度（即NAS modem的发
	      送速度），后面跟一个反斜杠（/），再跟接收速度，然后是其它任选信
	      息。
	     
	
	 
	
	
	Rigney, et al.               Informational                     [Page 30]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      如 “28800 V42BIS/LAPM”或“52000/31200 V90”。
	      
	      在计费请求报文中可以（may）包含超过一个Connect-Info属性，以－－－－－－－－－？???
	      便让modems用一种标准的格式报告更多的连接信息，信息的长度可能超过
	      252字节。
	
	
	5.12.  Configuration-Token
	
	   描述
	
	      这个属性应用在基于代理的大型分布式认证网络中。此属性由RADIUS代理服
	      务器放在接入成功回应报文中发送给RADIUS代理客户端，用来指明使用的
	      用户profile类型。此属性不应该发送给NAS。
	
	
	   Configuration-Token属性的格式如下所示。各个域是按照自左向右的顺序传输
	   的。
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |  String ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      78 代表Configuration-Token.
	
	   长度
	
	      >= 3
	
	   字符串
	
	      字符串域占位1个或者多个字节，该信息的实际格式由所使用的场所或者应
	      用程序决定，一个可靠的实现应该（SHOULD）将该域做为普通的字节对待。
	
	      该域允许的用法的定义超出了本规范的讨论范围。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 31]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	5.13.  EAP-Message
	
	   描述
	
	      本属性用于封装扩展认证协议[3]报文，以便让NAS即使不理解EAP协议，也
	      能利用EAP对拨号接入用户进行认证。
	
	      NAS把从用户接收到的EAP报文放在一个或多个EAP属性中，作为接入请求报
	      文的一部分转发给RADIUS服务器，RADIUS服务器可以在接入挑战报文，接入
	      成功回应报文或接入拒绝回应报文中返回EAP属性。
	
	      RADIUS服务器如果对收到的EAP报文不理解，应该（SHOULD）返回一个接入
	      拒绝回应报文。
	
	      NAS把从认证对等端接收到的EAP报文放在一个或多个EAP-Message属性中，
	      放在接入请求报文中转发给RADIUS服务器。如果接入请求报文或接入挑战报
	      文中包含多个EAP-Messages属性，那么它们必须（MUST）按连续地按照顺序
	      排好放在接入请求报文或接入挑战报文中。而接入成功回应报文和接入拒绝
	      回应报文中应该（SHOULD）只能有一个EAP-Message属性，此属性中包含了
	      EAP-Success或EAP-Failure。
	
	      希望用EAP实现多种认证方法，包括强壮的加密技术。为了防止攻击者通过
	      攻击RADIUS/EAP攻击EAP（例如，通过修改EAP-Success或EAP-Failure报
	      文），RADIUS/EAP有必要提供完整的保护，至少应该象EAP方法本身那样强
	      壮。
	
	      因此，必须用Message-Authenticator属性保护所有携带了EAP-Message属性
	      的接入请求报文，接入挑战报文，接入成功回应报文和接入拒绝回应报文。
	
	      对于携带有EAP-Message属性，但没有携带Message-Authenticator属性的接
	      入请求报文，RADIUS服务器应该（SHOULD）静默丢弃该报文。支持
	      EAP-Message属性的RADIUS服务器必须（MUST）能够计算出
	      Message-Authenticator属性的正确值，如果计算出的值与发送的值不匹
	      配，RADIUS服务器必须（MUST）丢弃该报文。如果RADIUS服务器不支持
	      EAP-Message属性，则当它收到一个含有EAP-Message属性的接入请求报文
	      时，必须（MUST）返回一个接入拒绝回应报文。如果RADIUS服务器收到一个
	      它不能理解的EAP-Message属性，也必须返回一个接入拒绝回应报文。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 32]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	      对于携带有EAP-Message属性的接入挑战报文，接入成功回应报文或接入拒
	      绝回应报文，若报文中不含有Message-Authenticator属性，则NAS应该丢弃
	      该报文。如果NAS支持EAP-Message属性，它必须（MUST）能够计算出
	      Message-Authenticator属性的正确值，如果计算出的值与发送的值不匹
	      配，NAS必须（MUST）丢弃此报文。
	
	 
	
	   EAP-Message属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     String...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      79 代表EAP-Message.
	
	   长度
	
	      >= 3
	
	   字符串
	
	      
	      字符串域包含了EAP报文，EAP报文在[3]中定义。如果报文中含有多个
	      EAP-Message属性，这几个属性应该（should）连续，因此在RADIUS协议中
	      允许长度大于253个字节的EAP报文。
	
	5.14.  Message-Authenticator
	
	   描述
	
	      本属性可以（MAY）用来对接入请求报文签名，以防止利用CHAP，ARAP或EAP
	      认证方法欺骗接入请求报文。此属性可以（MAY）用在任何接入请求报文
	      中，但必须（MUST）用在任何携带有EAP-Message属性的接入请求报文，接
	      入成功回应报文，接入拒绝回应报文或者接入挑战报文中。
	
	      如果RADIUS服务器接收到的接入请求报文中有Message-Authenticator属
	      性，那么服务器必须计算出正确的Message-Authenticator属性值，如果计
	      算出的值与发送的值不同，必须（MUST）静默丢弃该报文。
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 33]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      如果RADIUS客户端接收到的接入成功回应报文，接入拒绝回应报文或接入挑
	      战报文中含有Message-Authenticator属性，那么客户端必须计算出正确的
	      Message-Authenticator属性值，如果计算出的值与发送的值不同，必须
	      （MUST）就静默丢弃该报文。
	
	      此备忘录的早期草案将此属性称为“Signature”，但是
	      Message-Authenticator更准确。但具体操作没有改变，只是名字改变了而
	      已。
	
	   Message-Authenticator属性的格式如下所示。各个域是按照自左向右的顺序传
	   输的。
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     String...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      80 代表Message-Authenticator
	
	   长度
	
	      18
	
	   字符串
	
	      如果出现在接入请求报文中，Message-Authenticator属性是整个接入请求
	      报文的HMAC-MD5 [9]校验和，包括类型，标识符，长度和认证字，其中使用
	      共享密钥作为密钥，过程如下：
	
	      Message-Authenticator = HMAC-MD5 (Type, Identifier, Length,
	      Request Authenticator, Attributes)
	
	      当计算校验和时，签名字符串应该被看作16个为0的字节。
	
	      对于接入挑战报文，接入成功回应报文和接入拒绝回应报文，使用了接入请
	      求报文中的请求认证字，计算方法如下：
	
	      Message-Authenticator = HMAC-MD5 (Type, Identifier, Length,
	      Request Authenticator, Attributes)
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 34]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      当计算校验和时，签名字符串应该被看作16个为0的字节，共享密钥作为
	      HMAC-MD5 hash的密钥。应该在计算回应认证字之前计算出来并插入到报文
	      中。
	
	      如果报文中有User-Password属性时，就不需要该属性了，但在其它认证类
	      型中，对于防止攻击是有用的。此属性是为了阻止攻击者安装一个
	      “伪装”的NAS，实施针对RADIUS服务器的在线字典攻击。此属性不能提供
	      对离线攻击的防护，如攻击者截获了包括CHAP挑战和响应的报文，然后实施
	      针对这些离线报文的字典攻击。
	
	      IP Security最终会使该属性变得没有必要，因此该属性应该（should）看
	      作是一个过渡方法。
	
	 
	
	 
	
	
	5.15.  ARAP-Challenge-Response
	
	   Description
	
	      This attribute is sent in an Access-Accept packet with Framed-
	      Protocol of ARAP, and contains the response to the dial-in
	      client's challenge.
	
	   A summary of the ARAP-Challenge-Response attribute format is shown
	   below.  The fields are transmitted from left to right.
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     Value...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	                                   |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   Type
	
	      84 for ARAP-Challenge-Response.
	
	   Length
	
	      10
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 35]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Value
	
	      The Value field contains an 8 octet response to the dial-in
	      client's challenge. The RADIUS server calculates this value by
	      taking the dial-in client's challenge from the high order 8 octets
	      of the ARAP-Password attribute and  performing DES encryption on
	      this value with the authenticating user's password as the key. If
	      the user's password is less than 8 octets in length, the password
	      is padded at the end with NULL octets to a length of 8 before
	      using it as a key.
	
	5.16.  Acct-Interim-Interval
	
	   描述
	
	      该属性表示一个特定会话中的两个计费更新报文的间隔秒数，该属性只能出
	      现在接入成功回应报文中。
	
	
	   Acct-Interim-Interval属性的格式如下所示。各个域是按照自左向右的顺序传
	   输的。
	
	   0                   1                   2                   3
	   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |          Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      85 代表Acct-Interim-Interval.
	
	   长度
	
	      6
	
	   值
	
	      值域包含一个特定会话中的由NAS发送的两个计费更新报文的间隔秒数，该
	      值必须不能（MUST NOT）小于60，该值不应该（SHOULD NOT）小于600，应
	      该（should）认真考虑该值对网络流量的影响。
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 36]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	5.17.  NAS-Port-Id
	
	   描述
	
	      本属性包含了一个标识认证用户的NAS的端口的文本串。此属性只能用在接
	      入请求报文和计费请求报文中。需要注意的是此处的端口是NAS物理连接意
	      义上的端口，而不是TCP协议或UDP协议的端口号。
	
	      如果NAS区分它的端口，那么在接入请求报文中应该携带NAS-Port或
	      NAS-Port-Id属性。NAS-Port-Id目的是给不能方便地给端口编号的NAS使
	      用。
	
	 
	
	
	   NAS-Port-Id属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     Text...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	
	   类型
	
	      87 代表NAS-Port-Id.
	
	   长度
	
	      >= 3
	
	   文本
	
	      文本域包含了端口的名称。名称包含UTF-8编码的10646 [8]字符。
	
	
	5.18.  Framed-Pool
	
	   描述
	
	      该属性包含了一个地址池的名称，该地址池应该（SHOULD）是用来给用户分
	      配地址的。如果NAS不支持多个地址池，NAS应该（SHOULD）忽略该属性。地
	      址池一般用于IP地址，但是如果NAS支持其它协议，地址池也可以用于其他
	      协议。
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 37]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   Framed-Pool属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |     String...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      88 代表Framed-Pool
	
	   长度
	
	      >= 3
	
	   字符串
	
	      字符串域包含了NAS上配置的地址池的名称。
	
	
	5.19.  属性表
	
	   下表可以作为关于在哪个报文里可能携带哪个属性的应用指南。
	   Acct-Input-Gigawords，Acct-Output-Gigawords，Event-Timestamp
	   和NAS-Port-Id属可以（may）在计费请求报文里出现0次或1次。Connect-Info
	   属性可以（may）在计费请求报文中出现0次以上，本文档中增加的其它属性必
	   须不能（MUST NOT）出现在计费请求报文中。
	
	
	Request  Accept  Reject  Challenge   #    Attribute
	0-1      0       0       0           70   ARAP-Password [Note 1]
	0        0-1     0       0-1         71   ARAP-Features
	0        0-1     0       0           72   ARAP-Zone-Access
	0-1      0       0       0-1         73   ARAP-Security
	0+       0       0       0+          74   ARAP-Security-Data
	0        0       0-1     0           75   Password-Retry
	0        0       0       0-1         76   Prompt
	0-1      0       0       0           77   Connect-Info
	0        0+      0       0           78   Configuration-Token
	0+       0+      0+      0+          79   EAP-Message [Note 1]
	0-1      0-1     0-1     0-1         80   Message-Authenticator [Note 1]
	0        0-1     0       0-1         84   ARAP-Challenge-Response
	0        0-1     0       0           85   Acct-Interim-Interval
	0-1      0       0       0           87   NAS-Port-Id
	0        0-1     0       0           88   Framed-Pool
	Request  Accept  Reject  Challenge   #    Attribute
	
	 
	
	Rigney, et al.               Informational                     [Page 38]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   [注解 1] 如果接入请求报文中包含User-Password，CHAP-Password或者
	   ARAP-Password或者一个或多个EAP-Message属性，那么报文中包含的上述
	   四种属性类型必须不能（MUST NOT）超过一个。如果报文中不包括上述四种属
	   性中的任何一个，那么报文中应该包含一个Message-Authenticator属性。如果
	   报文中包含一个EAP-Message属性，那么报文中必须（MUST）包含
	   Message-Authenticator属性。
	
	   下表说明上表各表项的含义：
	
	      0     该属性必须不能（MUST NOT）出现在该类型报文中。
	      0+    0个或者多个该属性的实例可以（MAY）出现在该类型报文中。
	      0-1   0个或者1个该属性的实例可以（MAY）出现在该类型报文中。
	      1     该属性必须且只能有（MUST）一个实例出现在该类型的报文中。
	
	6.  IANA事项
	
	   在RFC 2865 [1]中“IANA事项”一节中描述的RADIUS命字空间中，IANA注册了
	   本文档定义的报文类型代码，属性类型以及属性值。和BCP 26 [10]一致。
	
	
	7.  安全事项
	
	   本文档中除了Message-Authenticator和EAP-Message属性外，其它属性没有超
	   出RFC 2865 [1]中已经说明的额外的安全考虑。
	
	 
	
	7.1.  Message-Authenticator安全
	
	   因为在NAS和RADIUS服务器之间使用了共享密钥，携带User-Password属性的接
	   入请求报文在的用户和发送接入请求的NAS之间建立了身份确认。携带
	   CHAP-Password或EAP-Message属性的接入请求报文没有User-Password属性，
	   因此为了和发送接入请求报文的NAS建立身份确认，应该（should）在没有携带
	   User-Password属性的接入请求报文中使用Message-Authenticator属性。
	  
	
	 
	
	7.2.  EAP Security
	
	   既然EAP的目的是为PPP认证提供增强的安全性，因此RADIUS安全地支持EAP是至
	   关重要的。特别是必须（must）解决下面的问题：
	
	      EAP服务器和PPP认证者分离
	      连接劫持
	      中间人攻击
	      多数据库
	
	 
	
	
	Rigney, et al.               Informational                     [Page 39]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	      协商攻击
	
	7.2.1.  EAP服务器和PPP认证者分离
	
	   EAP端点相互认证，协商加密组，为随后PPP加密取得一个会话密钥是可能的。
	
	   如果对端和EAP客户在同一台机器上，这不是一个问题。所需要的就是EAP客户
	   模块向PPP加密模块传递一个会话密钥。
	
	   当EAP和RADIUS一起使用时，情况就比较复杂了，因为一般PPP认证者和EAP服务
	   器不在同一台机器上。例如，EAP服务器可能是一台后端安全服务器，或者
	   RADIUS服务器上的一个模块。
	
	   在EAP服务器和PPP认证者在不同的机器上的情况下，关于安全有如下几方面的
	   含义。首先，交互认证将在对端和EAP服务器上发生，而不是在对端和认证者之
	   间。这意味着对端不可能证实它连接的NAS或隧道服务器的身份。
	
	   如前所述，当EAP/RADIUS被用来封装EAP报文时，从NAS或隧道服务器发往
	   RADIUS服务器的EAP/RADIUS接入请求报文中需要Message-Authenticator属性。
	   因为Message-Authenticator属性包含一个HMAC-MD5 hash项，使得RADIUS服务
	   器有可能验证接入请求报文的完整性和NAS或隧道服务器的身份。类似地，从
	   RADIUS服务器发往NAS的接入挑战报文也被验证并由HMAC-MD5 hash项保护完整
	   性，使得NAS或隧道服务器可以判定报文的完整性并且验证RADIUS服务器的身
	   份。此外，通过包含其它完整性保护方法来发送的EAP报文不能被欺诈的NAS或
	   隧道服务器成功地修改。
	
	   EAP服务器和PPP认证者在不同的机器上引来的第二个问题是，对端和EAP服务器
	   协商的会话密钥需要传送到认证者。因此必须提供从EAP服务器到要使用密钥的
	   认证者或隧道服务器的传送会话密钥的一个机制。这个传送机制的规范超出了
	   本文档的范围。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 40]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	7.2.2.  连接劫持
	
	   在这种攻击方式中，攻击者试图在NAS和RADIUS服务器之间或RADIUS服务器和后
	   端安全服务器之间的会话中注入一些报文。同RFC 2865 [1]中描述的那样，
	   RADIUS不支持加密，只有接入回应报文和接入挑战报文受完整性保护。此外，
	   在RFC 2865 [1]中描述的完整性保护机制比一些EAP方法使用的弱，使得攻击
	   EAP/RADIUS有可能扰乱那些方法。
	
	   如前所述，为了给EAP互换的所有报文提供验证，所有EAP/RADIUS报文必须使用
	   Message-Authenticator属性来验证。
	
	 
	
	 
	
	7.2.3.  中间人攻击
	
	   因为RADIUS安全基于共享密钥，在认证或计费报文通过一个代理链转发的情况
	   下不能提供端到端的安全。结果，获得一个RADIUS代理控制权的攻击者将能够
	   修改传送中的EAP报文。
	
	 
	
	7.2.4.  多数据库
	
	   在许多情况下，为了提供EAP服务，后端安全服务器和RADIUS服务器一块使用。
	   除非后端安全服务器也实现RADIUS服务器的功能，否则要用两个分离的用户数
	   据库，每一个数据库内包含有关用户安全需求的信息。这意味着安全的降低，
	   因为只要成功攻击数据库中的任何一个或攻击它们的后端数据库都可能损害安
	   全性。当使用多个用户数据库时，如果要增加一个用户，就必须进行多个操
	   作，这就增加了出错的概率。当用户信息也放在LDAP上时，危害性又被进一步
	   放大，因为此时必须在三个地方存放用户信息。
	
	   为了解决这些威胁，建议合并数据库。这可以通过以下途径来解决：RADIUS服
	   务器和后端安全服务器在同一个后端服务器上存储信息；后端服务器提供一个
	   完全的RADIUS实现；或将后端安全服务器和RADIUS服务器合并到同一个机器
	   上。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 41]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	7.2.5.  协商攻击
	
	   在协商攻击中，欺骗的NAS、隧道服务器、RADIUS代理或RADIUS服务器使认证对
	   等端选择一个安全小的方法，这样欺骗者可以比较容易地得到用户密码。例
	   如，一个会话本来在通常用EAP认证，被协商攻击后，可能用CHAP或PAP认证；
	   一个会话本来用一种EAP类型认证，被协商攻击后，可能使用一种安全系数小的
	   EAP类型认证，如MD5。以前认为很遥远的欺骗设备威胁，已经给电话公司交换
	   系统造成了危害，如在[11]中描述的。
	
	   要使系统不受协商攻击，需要消除向下协商。这可以通过以下方式实现：对认
	   证对等端而言，使用每一个连接策略，对RADIUS服务器而言，使用每一个用户
	   策略。
	
	   对于认证对等端而言，认证策略应该建立在每一个连接的基础之上。每一个连
	   接策略允许呼叫一种服务时认证对等端来协商EAP，而呼叫另一种服务时协商
	   CHAP，即使两种服务通过同一个电话号码就可以访问。
	
	   对于每一个连接策略，只有当期望支持EAP时，认证对等端才会试图协商EAP。
	   因此假定如果认证对等端选择EAP，那么就认为认证对等端需要哪个等级的安
	   全。如果不能提供，那么极有可能是配置错误，甚至认证对等端连接到了错误
	   的服务器上。万一NAS不能协商EAP，或NAS发送的EAP-Request与期望的不同，
	   认证对等端必须端开连接。期望协商EAP的认证对等端一定不能协商CHAP或
	   PAP。
	
	
	   NAS而言，在它知道用户的身份之前，它可能无法确定一个用户是否需要EAP认
	   证。例如，共用NAS，可能对一个专卖者实现EAP，对另外一个则不实现。在这
	   些情况下，如果NAS的每一个用户必须用EAP，那么NAS必须试图为每一个呼叫协
	   商EAP，这样可以避免能够EAP认证的客户端因用多种认证而削弱安全性。
	
	   如果协商了CHAP，NAS将把User-Name和CHAP-Password属性放在接入请求报文中
	   传给RADIUS服务器。如果没有要求用户用EAP认证，那么RADIUS服务器可以用接
	   入成功回应报文或接入拒绝回应报文响应，这要看用哪一个报文回应更合适。
	   但是，如果已经协商过了CHAP而需要EAP，那么RADIUS服务器必须回一个接入拒
	   绝回应报文，而不能回应Access-Challenge/EAP-Message/EAP-Request报文。
	   认证对等端必须拒绝重新协商认证，即使是从CHAP协商到EAP。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 42]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	 
	
	   如果已经协商过了EAP，但RADIUS代理或服务器不支持，那么服务器或代理必须
	   用接入拒绝回应报文响应。在这些情况下，NAS必须发送一个LCP-Terminate并
	   且切断用户。这是正确的行为，因为认证对等端期望协商EAP，但期望不能被满
	   足。对于支持EAP的认证对等端，如果最初已经协商了EAP，那么解决重新协商
	   认证协议。需要注意的是因RADIUS代理服务器不支持EAP而出现的问题是很难诊
	   断的。因为从一个地方拨号接入的用户（代理支持EAP）或许能够用EAP成功认
	   证，而同一个用户从另一个地方拨号进入时（代理不支持EAP），或许始终被切
	   断。
	
	 
	
	 
	
	 
	
	 
	
	
	8.  References
	
	   [1]  Rigney, C., Willens, S., Rubens, A. and W. Simpson, "Remote
	        Authentication Dial In User Service (RADIUS)", RFC 2865, June
	        2000.
	
	   [2]  Rigney, C., "RADIUS Accounting", RFC 2866, June 2000.
	
	   [3]  Blunk, L. and J. Vollbrecht, "PPP Extensible Authentication
	        Protocol (EAP)", RFC 2284, March 1998.
	
	   [4]  Bradner, S., "Key words for use in RFCs to Indicate Requirement
	        Levels", BCP 14, RFC 2119, March, 1997.
	
	   [5]  Reynolds, J. and J. Postel, "Assigned Numbers", STD 2, RFC 1700,
	        October 1994.
	
	   [6]  Zorn, G., Leifer, D., Rubens, A., Shriver, J., Holdrege, M.  and
	        I. Goyret, "RADIUS Attributes for Tunnel Protocol Support", RFC
	        2868, June 2000.
	
	   [7]  Zorn, G., Aboba, B. and D. Mitton, "RADIUS Accounting
	        Modifications for Tunnel Protocol Support", RFC 2867, June 2000.
	
	   [8]  Yergeau, F., "UTF-8, a transformation format of ISO 10646", RFC
	        2279, January 1998.
	
	 
	
	 
	
	
	Rigney, et al.               Informational                     [Page 43]
	
	RFC 2869                   RADIUS Extensions                   June 2000
	
	
	   [9]  Krawczyk, H., Bellare, M. and R. Canetti, "HMAC: Keyed-Hashing
	        for Message Authentication", RFC 2104, February 1997.
	
	   [10] Alvestrand, H. and T. Narten, "Guidelines for Writing an IANA
	        Considerations Section in RFCs", BCP 26, RFC 2434, October 1998.
	
	   [11] Slatalla, M., and  Quittner, J., "Masters of Deception."
	        HarperCollins, New York, 1995.
	
	9.  Acknowledgements
	
	   RADIUS and RADIUS Accounting were originally developed by Livingston
	   Enterprises (now part of Lucent Technologies) for their PortMaster
	   series of Network Access Servers.
	
	   The section on ARAP is adopted with permission from "Using RADIUS to
	   Authenticate Apple Remote Access Connections" by Ward Willats of Cyno
	   Technologies (ward@cyno.com).
	
	   The section on Acct-Interim-Interval is adopted with permission from
	   an earlier work in progress by Pat Calhoun of Sun Microsystems, Mark
	   Beadles of Compuserve, and Alex Ratcliffe of UUNET Technologies.
	
	   The section on EAP is adopted with permission from an earlier work in
	   progress by Pat Calhoun of Sun Microsystems, Allan Rubens of Merit
	   Network, and Bernard Aboba of Microsoft.  Thanks also to Dave Dawson
	   and Karl Fox of Ascend, and Glen Zorn and Narendra Gidwani of
	   Microsoft for useful discussions of this problem space.
	
	10.  Chair's Address
	
	   The RADIUS working group can be contacted via the current chair:
	
	   Carl Rigney
	   Livingston Enterprises
	   4464 Willow Road
	   Pleasanton, California  94588
	
	   Phone: +1 925 737 2100
	   EMail: cdr@telemancy.com
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney, et al.               Informational                     [Page 44]