	
	
	
	
	
	Network Working Group                                          C. Rigney
	Request for Comments: 2866                                    Livingston
	Category: Informational                                        June 2000
	Obsoletes: 2139
	翻译：twingao
	
	                           RADIUS Accounting
	
	备忘录状态
	
	   本文不是在制定一个Internet标准，只是向互联网社区提供相关信息，本文可
	   以不受限制地传播。
	
	版权说明
	
	   Copyright (C) The Internet Society (2000).  All Rights Reserved.
	
	
	摘要
	
	   本文描述了一个在网络接入服务器（Network Access Server）和共享的计费服
	   务器之间传送计费信息的协议。
	
	
	实现本协议的注意事项
	
	   本备忘录说明了RADIUS计费协议。早期开发的RADIUS记帐协议使用的是端口号
	   为1646的UDP端口，但由于它和著名的"sa-msg-port"服务互相冲突。所以官方
	   分配的RADIUS计费端口修改为1813。
	
	
	目录
	
	   1.     简介 ............................................    2
	     1.1    描述文档的约定 ................................    3
	     1.2    术语 ..........................................    3
	   2.     操作 ............................................    4
	     2.1    Proxy(代理) ...................................    4
	   3.     报文格式 ........................................    5
	   4.     报文类型 ........................................    7
	     4.1    Accounting-Request(计费请求) ..................    8
	     4.2    Accounting-Response(计费回应) .................    9
	   5.     Attributes(属性) ................................   10
	     5.1    Acct-Status-Type ..............................   12
	     5.2    Acct-Delay-Time ...............................   13
	     5.3    Acct-Input-Octets .............................   14
	     5.4    Acct-Output-Octets ............................   15
	     5.5    Acct-Session-Id ...............................   15
	
	 
	
	Rigney                       Informational                      [Page 1]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	     5.6    Acct-Authentic ................................   16
	     5.7    Acct-Session-Time .............................   17
	     5.8    Acct-Input-Packets ............................   18
	     5.9    Acct-Output-Packets ...........................   18
	     5.10   Acct-Terminate-Cause ..........................   19
	     5.11   Acct-Multi-Session-Id .........................   21
	     5.12   Acct-Link-Count ...............................   22
	     5.13   属性列表 ......................................   23
	   6.     IANA事项 ........................................   25
	   7.     安全事项 ........................................   25
	   8.     更改记录 ........................................   25
	   9.     参考文献 ........................................   26
	   10.    致谢 ............................................   26
	   11.    AAA工作组主席地址 ...............................   26
	   12.    作者地址 ........................................   27
	   13.    版权声明 ........................................   28
	
	1.  简介
	
	   要经营为众多的用户提供的串口线路和modem池，这会带来巨大的管理支持方面
	   的需求。由于modem池是通向外部的链路，因此它对安全、认证、计费都提出了
	   很高的要求。可以通过维护一个用户数据库来实现该需求，该数据库包含了认
	   证（验证用户的名字和密码）以及为用户提供的服务类型的详细的配置信息
	   （如SLIP,PPP,telnet,rlogin等）。
	
	   RADIUS(远程用户拨号认证系统)文档[2]详细说明了RADIUS协议中认证和授权的
	   相关内容。本文扩展了RADIUS协议的应用，说明了从NAS（Network Access
	   Server）到RADIUS服务器的计费信息传递的协议。
	
	   本文废弃了RFC 2139 [1]。它与RFC 2139之间的差别可以在附录“更改记录”
	   中找到。
	
	 
	
	 
	
	 
	
	
	   RADIUS计费协议的主要特性：
	
	      客户/服务器模式
	          网络接入服务器(NAS)是RADIUS计费服务器的客户端。客户端负责将用
	          户的计费信息传递给指定的RADIUS计费服务器。
	
	 
	
	 
	
	 
	
	
	Rigney                       Informational                      [Page 2]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	          RADIUS计费服务器负责接收计费请求，并给客户端返回一个回应信息，
	          表示成功接收到了计费请求。
	          
	          RADIUS计费服务器也可以作为其他类型的计费服务器的代理。
	
	 
	
	      网络安全
	          
	          客户端与RADIUS计费服务器之间的交互是通过共享密钥来进行相互认证
	          的，共享密钥不会通过网络传送。
	
	
	      协议扩充性
	          
	          所有的报文交互都是由多个不同长度的属性-长度-值三元组
	          （Attribute-Length-Value 3元组）构成的。新的属性值的加入不会影
	          响到原有协议的实现。
	
	1.1.  描述文档的约定
	
	   本文中的关键词"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
	   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", 以及"OPTIONAL"的参见
	   RFC 2119 [3]中的描述。这些关键词意义与其是否大写无关。
	
	
	1.2.  术语
	
	   本文使用了以下的术语：
	
	   服务      NAS为拨入用户提供的某种服务，如：PPP或者Telnet。
	  
	
	   会话      NAS为拨入用户提供的每一个服务都会建立一个会话。第一次开始提
	             供服务做为会话的开始，服务终止做为会话的结束。如果NAS支持的
	             话，一个用户可以有多个并行或者串行的会话。对于每一个会话，
	             都会产生一个独立的计费开始和计费结束记录，它们以不同的
	             Acct-Session-ID属性值区分。
	  
	
	 
	
	   静默丢弃  NAS或RADIUS服务器不对报文进行任何处理就直接丢弃。NAS或
	             RADIUS服务器应该(SHOULD)提供记录错误的能力，其中包括被静默
	             丢弃的报文的内容，而且应该(SHOULD)在一个统计计数器中记录下
	             该事件。
	
	
	
	
	Rigney                       Informational                      [Page 3]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	2.  操作
	
	   当一个客户端被配置成采用RADIUS计费协议时，在开始提供服务的时候它会生
	   成一个计费开始报文，报文描述了服务类型以及被服务的用户的信息，该报文
	   被发送到RADIUS计费服务器。计费服务器会返回应答，表示计费报文已经收
	   到。服务终止时，客户端会产生一个计费结束报文，该报文描述了服务类型以
	   及一些可选的统计数据，譬如，服务总时长、输入和输出的字节数或者输入和
	   输出报文数。该报文被发送到RADIUS计费服务器，计费服务器会返回应答，表
	   示计费报文已经收到。
	
	 
	
	 
	
	   计费请求（不管是计费开始报文还是计费结束报文）通过网络发送给RADIUS计
	   费服务器。本文推荐（recommended）客户端应该反复发送计费请求，直到收到
	   回应消息为止。如果在一段时间内没有收到回应消息，计费请求就会被重发几
	   次。如果主服务器宕机或者是无法到达，客户端也可以向备用服务器转发计费
	   请求，可以在对主服务器重试失败一定的次数之后客户端将计费请求转发到备
	   用服务器上，也可以采用循环方式将计费请求转发到备用服务器上。重试和放
	   弃算法是目前正在研究的一个课题，在本文中就不再赘述了。
	
	 
	
	 
	
	   RADIUS计费服务器可以（MAY）向其他的服务器转发计费请求，这时该RADIUS服
	   务器是一个客户端。
	
	   如果RADIUS计费服务器不能成功的记录计费报文，服务器一定不能（MUST NOT）
	   给客户端发送计费回应报文。
	
	
	2.1.  代理
	
	   参阅“RADIUS” RFC [2]中关于代理RADIUS的描述，代理RADIUS计费服务器
	   的工作方式与代理RADIUS服务器是相同的，如下例所示：
	
	
	   1.    NAS发送计费请求报文给转发服务器。
	
	   2.    转发服务器将计费请求记录下来（如果必要的话），在所有其它的
	         Proxy-State属性之后添加自己的Proxy-State属性（如果必要的话），
	         并且更新请求Authenticator（认证字），然后将计费请求转发给远程服
	         务器。
	
	 
	
	 
	
	
	Rigney                       Informational                      [Page 4]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   3.    远程服务器将计费请求记录下（如果必要的话），将所有的Proxy-State
	         属性按顺序原封不动的从请求数据报文复制到回应报文中，然后将计费
	         回应报文发送给转发服务器。
	  
	
	   4.    转发服务器剥离最后一个Proxy-State属性（如果在第2步添加了的
	         话），更新回应Authenticator（认证字），然后将计费回应报文发送给
	         NAS。
	
	   转发服务器一定不能（MUST not）更改当前报文中已经存在的Proxy-State或者
	   Class属性。
	
	   转发服务器可以（MAY）以通过（pass through）方式实现转发功能，这样，转
	   发服务器只要收到重发报文就将该报文转发；或者转发服务器可以（MAY）自身
	   实现重发功能，这一般在转发服务器和远程服务器之间的网络链路与NAS和转发
	   服务器之间的链路相比有很大不同的情况下。
	
	
	
	   当代理服务器实现重发功能的时候，要特别的注意确保重发算法是健壮的和可
	   扩展的。
	
	
	3.  报文格式
	
	   准确地讲，RADIUS计费报文被封装在UDP报文的数据域[4]，它的UDP目的端口号
	   是1813（十进制数）。
	
	
	   当回应报文的时候，源端口和目的端口互换。
	
	
	   本文定义了RADIUS计费协议。早期的RADIUS计费协议使用UDP端口1646，由于它
	   和著名的"sa-msg-port"服务相冲突。官方为RADIUS计费协议重新分配的端口号
	   是1813。
	
	
	   RADIUS报文格式如下所示。各个域的数据是从左向右传输的。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney                       Informational                      [Page 5]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Code      |  Identifier   |            Length             |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                                                               |
	   |                         Authenticator                         |
	   |                                                               |
	   |                                                               |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |  Attributes ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-
	
	
	   Code(代码)                                                            --------????????
	
	      Code域占位一个字节，它用来标识RADIUS报文类型。当收到的报文Code域非
	      法时，该报文将会被静默丢弃。
	
	      RADIUS计费报文Code域（十进制）分配如下：
	
	           4      计费请求
	           5      计费回应
	
	
	   Identifier（标识符）
	
	      Identifier域占位一个字节，用于匹配请求和回应报文。如果在一个很短的
	      时间内接收到相同的源IP地址、源UDP端口号和相同的Identifier域的请求
	      报文，RADIUS服务器就可以认为是重复的请求报文。
	
	
	   Length（长度）
	
	      Length域占位两个字节。它包含了报文中的Code域，Identifier域，
	      Length域，Authenticator域和属性域的总长度。在长度域限定的范围之外
	      的字节必须（MUST）作为填充字节，在接收时不予处理。如果报文的实际长
	      度小于长度域中给出的值，该报文必须（MUST）被静默丢弃。报文的最小长
	      度是20，最大长度是4095。
	
	 
	
	   Authenticator（认证字）
	
	      Authenticator域占位16个字节。高位字节先传输。该域的值用来鉴别客户
	      端和RADIUS计费服务器之间的消息。
	  
	
	 
	
	
	Rigney                       Informational                      [Page 6]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   Request Authenticator（请求认证字）
	
	      在计费请求报文中，认证字的值是一个占位16个字节的MD5 [5]校验和，称
	      作请求认证字。
	
	      NAS和RADIUS计费服务器共享一个密钥。计费请求报文中的认证字中包含对
	      一个由Code+Identifier+Length+16个值为0的字节+请求属性+共享密钥
	      （+表示将各个字符连接起来）所构成的字节流进行某种方式的MD5哈希计算
	      得出的16个字节的hash值。这个占位16个字节的MD5哈希值被存储到计费请
	      求包的Authenticator域中。
	
	      注意计费请求中的请求认证字不得与RADIUS接入请求的请求认证字的生成方
	      式相同，因为在计费请求中没有User-Password属性。
	
	 
	
	 
	
	   Response Authenticator（回应认证字）
	
	      在计费回应报文中的Authenticator域称作回应认证字。它包含对一个由计
	      费回应Code+Identifier+Length+对应的计费请求包的请求认证字+回应属性
	      （如果有的话）+共享密钥构成的字节流进行某种方式的MD5哈希计算得出的
	      16个字节的hash值。这个占位16个字节的MD5哈希值被存储到计费回应报文
	      的Authenticator域中。
	
	 
	
	
	   属性
	
	   属性可以（may）包含多个实例，在这种情况下，同种类型的各个属性应当
	   （SHOULD）有一定的顺序。但是，不同类型的各个属性不需要有顺序。
	
	
	4.  报文类型
	
	   RADIUS报文类型是由位于报文的第一个字节的Code域决定的。
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	Rigney                       Informational                      [Page 7]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	4.1.  计费请求
	
	   描述
	
	      计费请求报文是由客户端（典型的情况是NAS或者代理服务器）发送到
	      RADIUS计费服务器的，其中包含为用户提供服务的计费的信息。客户端发送
	      一个Code域值为4（计费请求）的RADIUS数据报文。
	
	      一旦接收到计费请求报文，如果服务器能够成功地记录下计费报文的话，必
	      须（MUST）回应一个计费回应报文，但如果记录计费报文失败，则必须不能
	      （MUST NOT）回应计费回应报文。
	      
	      除了User-Password, CHAP-Password, Reply-Message, State属性外
	      （MUST NOT），其它在接入请求和接入成功回应报文中有效的属性，在计费请求
	      报文中同样有效。在RADIUS计费请求报文中必须（MUST）包含
	      NAS-IP-Address或者NAS-Identifer属性。在计费请求报文中还应当
	      （SHOULD）包含NAS-port或者NAS-Port-Type属性，或者两者都包含，除非
	      服务不涉及端口或者NAS不区分端口。
	
	
	      如果计费请求报文包含Framed-IP-Address属性，则该属性必须（MUST）是
	      用户IP地址。如果接入成功回应报文中包含了有具体值的
	      Framed-IP-Address属性，告诉NAS需要给用户分配或者与用户协商一个IP地
	      址，则计费请求报文中的Framed-IP-Address属性（如果有的话）必须是分
	      配的或者协商的实际的IP地址。
	
	 
	
	 
	
	
	   计费请求报文格式如下所示。
	   
	   各个域是自左向右传输的。
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Code      |  Identifier   |            Length             |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                                                               |
	   |                     Request Authenticator                     |
	   |                                                               |
	   |                                                               |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |  Attributes ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-
	
	 
	
	
	Rigney                       Informational                      [Page 8]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   Code（代码）
	
	      4 代表计费请求报文
	
	   Identifier（标识符）
	
	      当属性域的内容发生改变或者是已经收到前一个请求的有效的回应，
	      Identifier域必须（MUST）改变。如果仅仅是由于重传而内容没有变化时，
	      标识符必须（MUST）保持不变。
	
	      需要指出的是，如果计费请求报文中包含了Acct-Delay-Time属性，则当该
	      报文重传时，Acct-Delay-Time属性值将被更新，这导致了属性域内容发生
	      变化，这时需要一个新的Identifier域和Request Authenticator域。
	
	 
	
	
	   Request Authenticator（请求认证字）
	
	      计费请求报文的请求认证字是一个占位16个字节的MD5哈希值，该值的计算
	      方法已在上述的“Request Authenticator”中描述了。
	
	
	   Attributes（属性）
	
	      属性域的长度是变化的，其中包含着一系列的属性。
	
	
	4.2.  计费回应
	
	   描述
	
	      计费回应报文是由RADIUS计费服务器发送给客户端的，用来通知客户端计费
	      请求报文已接收到，并且成功地记录下来了。如果计费请求报文被成功地记
	      录下来，RADIUS计费服务器必须（MUST）发送一个Code域值为5（计费回
	      应）的报文。客户端收到计费回应报文后，其Identifier域需要能够和一个
	      等待回应的计费请求报文的Identifier域相匹配。回应认证字域必须
	      （MUST）含有对等待回应的的计费请求的正确响应，而无效的数据包会被静
	      默丢弃。
	
	      RADIUS计费回应报文不要求包含属性。
	
	 
	
	
	   计费回应报文格式如下所示。各个域是自左向右传输的。
	
	 
	
	 
	
	Rigney                       Informational                      [Page 9]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Code      |  Identifier   |            Length             |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                                                               |
	   |                     Response Authenticator                    |
	   |                                                               |
	   |                                                               |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |  Attributes ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-
	
	   Code（代码）
	
	      5 代表计费回应
	
	   Identifier（标识符）
	
	      Identifier域是对引起这次回应的计费请求的Identifier域的一个拷贝。
	
	
	   Response Authenticator（回应认证字）
	
	      计费回应报文的Response Authenticator域是一个占位16个字节的MD5哈希
	      值，该值的计算方法已在上述的“Response Authenticator”中描述了。
	
	
	   Attributes（属性）
	
	      Attributes域的长度是变化的，其中包含了零个或者多个属性。
	
	
	5.  属性
	
	   RADIUS属性通过请求报文和回应报文携带了关于的认证、授权和计费方面的信
	   息细节。
	
	   有些属性可以（MAY）被多次包含。这通常有特别的目的，这种情况都会在每个
	   属性的描述中特别指出。
	
	   是由RADIUS报文的Length域来决定属性列表在何处结束。
	
	   属性域的格式如下所示。是自左向右传输的。
	
	 
	
	 
	
	 
	
	Rigney                       Informational                     [Page 10]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |  Value ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	
	   Type（类型）
	
	      Type占位一个字节。到目前为止，在最新的“Assigned Number” RFC [6]
	      中给出了RADIUS Type的值的详细描述。值192-223是保留给实验用的，值
	      224-240是保留给特定实现用的，值241-255是预留的，而且不应该（should
	      not）使用它们。本文中涉及到以下的Type值：
	
	           1-39  （参见RADIUS文档 [2]）
	          40      Acct-Status-Type
	          41      Acct-Delay-Time
	          42      Acct-Input-Octets
	          43      Acct-Output-Octets
	          44      Acct-Session-Id
	          45      Acct-Authentic
	          46      Acct-Session-Time
	          47      Acct-Input-Packets
	          48      Acct-Output-Packets
	          49      Acct-Terminate-Cause
	          50      Acct-Multi-Session-Id
	          51      Acct-Link-Count
	          60+ （参考RADIUS文档 [2]）
	         
	
	
	   Length（长度）
	
	      Length域占位一个字节，表示包括Type、Length、Value域在内的属性的长
	      度。如果接受到的计费请求报文中的属性长度无效，整个请求报文必须
	      （MUST）静默丢弃。
	
	
	   Value（值）
	
	      Value域占位零个或者更多字节，包含属性信息的详细描述。值域的格式和
	      长度是由属性的类型和长度决定的。
	
	      需要指出的是，在RADIUS中没有任何类型的属性值是以NUL（十六进制的
	      0x00）结束的。譬如，“text”和“string"类型的属性值是不能以NUL结
	      束。由于属性具有长度域，因而不必使用结束符。“text”类型的属性值包
	      含UTF-8编码的10646 [7]字符，而“string”类型的属性值含有8位二进制
	
	 
	
	
	Rigney                       Informational                     [Page 11]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	      数据。服务器和客户端必须（MUST）能够处理嵌入的null。使用C语言实现
	      的RADIUS在处理字符串时需要注意不能使用strcpy()函数。
	
	 
	
	      值域有五种数据类型。注意：类型“text”是类型“string"的一个子集。
	
	      text     1-253个字节，包含UTF-8编码的10646 [7]字符。长度为零的text
	               类型的属性必须不能（MUST NOT）被发送，而应该将整个属性忽、
	               略。
	
	      string   1-253个字节，包含二进制数据（值从0到255，十进制）。长度为
	               零的string类型的属性必须不能（MUST NOT）发送，而应该将整
	               个属性忽略。
	
	      address  32位的数值，高位字节优先传输。
	
	      integer  32位的无符号数，高位字节优先传输。
	
	      time     32位的无符号数，高位字节优先传输。从格林威治时间1970年1月
	               1日0时0分0秒时起的秒数。标准的属性是不使用该数据类型的，
	               在这里提到该数据类型主要是有可能在将来的属性中使用。
	
	 
	
	
	5.1.  Acct-Status-Type
	
	   描述
	
	      该属性表明当前的计费请求报文是表示用户服务开始(Start)还是结束
	      （Stop）。
	
	      它可能（MAY）被客户端通过指定计费开始（标志）的方式来表示开始计费
	      （例如：在启动的时候），或者通过指定计费结束（标志）的方式来表示结
	      束计费（例如：在预定的重启之前）。
	
	
	   Acct-Status-Type属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	Rigney                       Informational                     [Page 12]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   Type（类型）
	
	      40 表示Acct-Status-Type
	
	   Length（长度）
	
	      6
	
	   Value（值）
	
	      值域占位四个字节。
	
	       1      Start（计费开始）
	       2      Stop（计费结束）
	       3      Interim-Update（计费更新）
	       7      Accounting-On（计费开始，通常为设备重启后）
	       8      Accounting-Off（计费结束，通常为设备重启前）
	       9-14   Reserved for Tunnel Accounting（为隧道计费保留）
	      15      Reserved for Failed（为计费失败保留）
	
	5.2.  Acct-Delay-Time
	
	   描述
	
	      该属性表明客户端试图发送该报文所延误的秒数。用该报文到达服务器端的
	      时间减去Acct-Delay-Time就可以知道生成该报文的大概时间（忽略网络传
	      输时间）。
	
	      注意，Acct-Delay-Time的改变会引起Identifier域的变化，参见上面关于
	      Identifier章节的讨论。
	
	 
	
	   Acct-Delay-Time属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	 
	
	 
	
	
	Rigney                       Informational                     [Page 13]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   类型
	
	      41 代表Acct-Delay-Time
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	5.3.  Acct-Input-Octets
	
	   描述
	
	      该属性表明在提供服务的过程中用户从端口接收到的字节总数。该属性只有
	      在计费结束请求报文中出现。
	
	 
	
	   Acct-Input-Octets属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      42 代表Acct-Input-Octets
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	 
	
	 
	
	 
	
	
	Rigney                       Informational                     [Page 14]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	5.4.  Acct-Output-Octets
	
	   描述
	
	      该属性表明在提供服务的过程中用户发送到端口的字节总数。该属性只有在
	      计费结束请求报文中出现。
	
	 
	
	   Acct-Output-Octets属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      43 代表Acct-Output-Octets
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	5.5.  Acct-Session-Id
	
	   描述
	
	      该属性是便于在日志文件中匹配计费开始和计费结束记录的唯一的计费ID。
	      对于一个给定的会话，计费开始和计费结束记录必须（MUST）有相同的
	      Acct-Session-Id。计费请求报文必须（MUST）有一个Acct-Session-Id。接
	      入请求报文可以（MAY）包含有Acct-Session-Id，如果接入请求报文中包含
	      Acct-Session-Id的话，在同一个会话中，NAS在计费请求报文中必须使用相
	      同的Acct-Session-Id。
	
	      Acct-Session-Id应当（SHOULD）是一个含有UTF-8编码的10646 [7]字符的
	      字符串。
	
	 
	
	 
	
	
	Rigney                       Informational                     [Page 15]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	      例如：有一种实现方式是使用一个8个数字的大写16进制数，该数的前两位
	      每次重新启动后加1（重启256次之后循环），后六位数字从0到2^24-1，大
	      约一千六百万个，用来计数重启之后登录的用户的个数。当然其它编码方式
	      也可以采用。
	
	
	   Acct-Session-Id属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |  Text ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      44 代表Acct-Session-Id
	
	   长度
	
	      >=3
	
	   字符串
	
	      字符串域应当（SHOULD）是一个含有UTF-8编码的10646 [7]字符的字符
	      串。
	
	5.6.  Acct-Authentic
	
	   描述
	
	      该属性可以（MAY）包含在计费请求报文中，用来说明用户的认证方式，是
	      RADIUS认证、NAS本地认证或者通过其它远程认证协议认证。如果一个用户
	      不需要认证就能够使用服务，就不应该（SHOULD NOT）生成计费记录。
	
	 
	
	   Acct-Authentic属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	Rigney                       Informational                     [Page 16]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   类型
	
	      45 表示Acct-Authentic
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	      1      RADIUS
	      2      Local
	      3      Remote
	
	5.7.  Acct-Session-Time
	
	   描述
	
	      该属性表明了用户接受服务的时间，该属性只能在计费结束报文中出现。
	
	 
	
	   Acct-Session-Time属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      46 代表Acct-Session-Time
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	 
	
	 
	
	Rigney                       Informational                     [Page 17]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	5.8.  Acct-Input-Packets
	
	   描述
	
	      该属性表明在提供服务的过程中用户从端口接收到的数据包总数。该属性只
	      有在计费结束请求报文中出现。
	
	 
	
	   Acct-Input-Packets属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      47 代表Acct-Input-Packets
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	5.9.  Acct-Output-Packets
	
	   描述
	
	      该属性表明在提供服务的过程中用户发送到端口的数据包总数。该属性只有
	      在计费结束请求报文中出现。
	
	 
	
	   Acct-Output-Packets属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	 
	
	 
	
	 
	
	 
	
	Rigney                       Informational                     [Page 18]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      48 代表Acct-Output-Packets
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节
	
	
	5.10.  Acct-Terminate-Cause
	
	   描述
	
	      该属性表明会话如何被终止的，该属性只有在计费结束请求报文中出现。
	
	 
	
	   Acct-Terminate-Cause属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	Rigney                       Informational                     [Page 19]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   类型
	
	      49 代表Acct-Terminate-Cause
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节。包含一个表示计费会话终止原因的整数。对应关系如
	      下：
	
	      1       User Request（用户请求）
	      2       Lost Carrier
	      3       Lost Service（服务丢失）
	      4       Idle Timeout（闲置超时）
	      5       Session Timeout（会话超时）
	      6       Admin Reset（管理员重置）
	      7       Admin Reboot（管理员重启）
	      8       Port Error（端口错误）
	      9       NAS Error（NAS错误）
	      10      NAS Request（NAS请求）
	      11      NAS Reboot（NAS重启）
	      12      Port Unneeded（端口不再需要）
	      13      Port Preempted（端口被抢占）
	      14      Port Suspended（端口挂起）
	      15      Service Unavailable（服务无法获得）
	      16      Callback（回调）
	      17      User Error（用户错误）
	      18      Host Request（主机请求）
	
	      会话终止原因如下：
	      
	      User Request         用户请求终止该项服务。例如：LCP终端或者用户退
	                           出。
	                           
	      Lost Carrier         DCD在端口处掉线。
	
	      Lost Service         无法再提供服务；例如：用户与主机之间的连接中
	                           断。
	
	
	      Idle Timeout         闲置时间超时
	
	      Session Timeout      最大会话时长超时
	
	      Admin Reset          管理员重置端口或者会话
	
	 
	
	Rigney                       Informational                     [Page 20]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	      Admin Reboot         管理员终止在NAS上的服务，例如：在重新启动NAS
	                           之前。
	
	      Port Error           由于NAS在端口上检测到错误，所以要求中止会话。
	
	
	      NAS Error            由于NAS检测到了错误（除了端口错误），所以要求
	                           终止会话。
	
	      NAS Request          NAS不是由于故障而要求中止会话，具体原因不在这
	                           里另外列举。
	
	      NAS Reboot           NAS终止会话，以进行非管理性的重启（系统崩溃）。
	
	
	      Port Unneeded        由于资源使用量低于最低水平线，NAS终止会话（例
	                           如：bandwidth-on-demand算法判定已经不再需要该
	                           端口了）。
	
	
	      Port Preempted       NAS终止会话以将端口分配给更高的优先级（服务）
	                           使用。
	
	      Port Suspended       NAS终止对话以挂起一个虚拟会话。
	
	
	      Service Unavailable  NAS无法提供要求的服务。
	
	      Callback             为了为新的会话执行回调操作，NAS终止当前的会
	                           话。
	
	      User Error           用户的输入有错误，导致中止会话。
	
	
	      Host Request         登录的主机正常终止会话。
	
	5.11.  Acct-Multi-Session-Id
	
	   描述
	
	      该属性做为一个唯一计费会话ID，通过该ID能够很容易将多个相互关联的会话
	      在日志文件中联系起来。被关联的每个会话都有各自唯一的
	      Acct-Session-Id，但它们有相同的Acct-Multi-Session-Id。强烈建议
	      （recommended）Acct-Multi-Session-Id包含UTF-8编码的10646 [7]字
	      符。
	
	   Acct-Multi-Session-Id属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	 
	
	
	Rigney                       Informational                     [Page 21]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	    0                   1                   2
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |  String ...
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	   类型
	
	      50 代表Acct-Multi-Session-Id
	
	   长度
	
	      >=3
	
	   字符串
	
	      字符串域应当（SHOULD）是一个含有UTF-8编码的10646 [7]字符的字符
	      串。
	
	5.12.  Acct-Link-Count
	
	   描述
	
	      该属性给出计费记录生成时该多链路会话的已经知道的链路个数。NAS在所
	      有可能含有多条链路的计费请求报文中都可以（MAY）包含Acct-Link-Count
	      属性。
	
	   Acct-Link-Count属性格式如下所示。各个域是按照自左向右的顺序传输的。
	
	
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	
	 
	
	 
	
	 
	
	 
	
	 
	
	 
	
	
	Rigney                       Informational                     [Page 22]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	   类型
	
	      51 代表Acct-Link-Count
	
	   长度
	
	      6
	
	   值
	
	      值域占位四个字节。表示在本多链路会话中目前所知道的链路的数目。
	
	      它可以用来使计费服务器知道到何时为止给定的多链路会话的所有记录都已
	      经接收到了。当计费服务器收到具有相同的Acct-Multi-Session-Id属性和
	      唯一Acct-Session-Id属性的计费结束请求报文个数等于所有这些计费结束
	      请求报文中Acct-Link-Count属性的最大值时，表示该多链路会话的所有计
	      费结束请求报文都已经收到了。
	
	      为了便于理解，下面给出一个8个计费请求报文的例子。为了清楚起见，只
	      给出了相关的属性，但是其他包含计费信息的属性也在计费请求中存在的。
	
	      Multi-Session-Id   Session-Id   Status-Type   Link-Count
	      "10"               "10"         Start         1
	      "10"               "11"         Start         2
	      "10"               "11"         Stop          2
	      "10"               "12"         Start         3
	      "10"               "13"         Start         4
	      "10"               "12"         Stop          4
	      "10"               "13"         Stop          4
	      "10"               "10"         Stop          4
	
	 
	
	 
	
	
	5.13.  属性列表
	
	   下表中给出了计费请求报文中可能会包含的属性。除了Proxy-State属性以及
	   （可能的）厂商自定义的属性之外，计费回应报文不应该包含其它属性。
	
	 
	
	
	                      #     Attribute
	                      0-1   User-Name
	                      0     User-Password
	                      0     CHAP-Password
	
	 
	
	Rigney                       Informational                     [Page 23]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	                      0-1   NAS-IP-Address [Note 1]
	                      0-1   NAS-Port
	                      0-1   Service-Type
	                      0-1   Framed-Protocol
	                      0-1   Framed-IP-Address
	                      0-1   Framed-IP-Netmask
	                      0-1   Framed-Routing
	                      0+    Filter-Id
	                      0-1   Framed-MTU
	                      0+    Framed-Compression
	                      0+    Login-IP-Host
	                      0-1   Login-Service
	                      0-1   Login-TCP-Port
	                      0     Reply-Message
	                      0-1   Callback-Number
	                      0-1   Callback-Id
	                      0+    Framed-Route
	                      0-1   Framed-IPX-Network
	                      0     State
	                      0+    Class
	                      0+    Vendor-Specific
	                      0-1   Session-Timeout
	                      0-1   Idle-Timeout
	                      0-1   Termination-Action
	                      0-1   Called-Station-Id
	                      0-1   Calling-Station-Id
	                      0-1   NAS-Identifier [Note 1]
	                      0+    Proxy-State
	                      0-1   Login-LAT-Service
	                      0-1   Login-LAT-Node
	                      0-1   Login-LAT-Group
	                      0-1   Framed-AppleTalk-Link
	                      0-1   Framed-AppleTalk-Network
	                      0-1   Framed-AppleTalk-Zone
	                      1     Acct-Status-Type
	                      0-1   Acct-Delay-Time
	                      0-1   Acct-Input-Octets
	                      0-1   Acct-Output-Octets
	                      1     Acct-Session-Id
	                      0-1   Acct-Authentic
	                      0-1   Acct-Session-Time
	                      0-1   Acct-Input-Packets
	                      0-1   Acct-Output-Packets
	                      0-1   Acct-Terminate-Cause
	                      0+    Acct-Multi-Session-Id
	                      0+    Acct-Link-Count
	                      0     CHAP-Challenge
	
	 
	
	
	Rigney                       Informational                     [Page 24]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	                      0-1   NAS-Port-Type
	                      0-1   Port-Limit
	                      0-1   Login-LAT-Port
	
	   [Note 1]计费请求报文中必须（MUST）包含NAS-IP-Address属性或者
	   NAS-Identifier（或者两者都有）。
	
	   下表中给出了上面的表格的说明：
	
	      0     该属性不能（MUST NOT）出现。
	      0+    该属性可以（MAY）出现0次或者多次。
	      0-1   该属性可以（MAY）出现0次或者1次。
	      1     该属性只能且必须（MUST）出现1次。
	
	6.  IANA事项
	
	   在本文中给出的报文类型代码、属性类型、属性值的定义都已经被IANA注册在
	   RADIUS名称空间，这已经在RFC 2865 [2]的“IANA事项”章节给出了详细的描
	   述。
	
	 
	
	7.  安全事项
	
	   安全问题已经在计费请求和计费回应中的关于认证字的章节中讨论过了。它采
	   用了一个从不在网络上传输的共享密钥。
	
	
	8.  更改记录
	
	   用UTF-8编码代替US-ASCII编码。
	
	   增加了关于代理的注释。
	
	   要求Framed-IP-Address属性应当（should）包含用户的真正的IP地址。
	
	   如果接入请求报文中包含了Acct-Session-ID属性，那么也必须（must）包含在
	   该会话的计费请求报文中。
	
	   在Acct-Status-Type属性中的添加了一些新值。
	
	   增加一个IANA事项的章节。
	
	   更新了参考文献。
	
	   将文本字符串定义成字符串的字集，更加明确了UTF-8编码的使用。
	
	 
	
	 
	
	Rigney                       Informational                     [Page 25]
	
	RFC 2866                   RADIUS Accounting                   June 2000
	
	
	9.  参考文献
	
	   [1]  Rigney, C., "RADIUS Accounting", RFC 2139, April 1997.
	
	   [2]  Rigney, C., Willens, S., Rubens, A. and W. Simpson, "Remote
	        Authentication Dial In User Service (RADIUS)", RFC 2865, June
	        2000.
	
	   [3]  Bradner, S., "Key words for use in RFCs to Indicate Requirement
	        Levels", BCP 14, RFC 2119, March, 1997.
	
	   [4]  Postel, J., "User Datagram Protocol", STD 6, RFC 768, August
	        1980.
	
	   [5]  Rivest, R. and S. Dusse, "The MD5 Message-Digest Algorithm", RFC
	        1321, April 1992.
	
	   [6]  Reynolds, J. and J. Postel, "Assigned Numbers", STD 2, RFC 1700,
	        October 1994.
	
	   [7]  Yergeau, F., "UTF-8, a transformation format of ISO 10646", RFC
	        2279, January 1998.
	
	   [8]  Alvestrand, H. and T. Narten, "Guidelines for Writing an IANA
	        Considerations Section in RFCs", BCP 26, RFC 2434, October 1998.
	
	10.  Acknowledgements
	
	   RADIUS and RADIUS Accounting were originally developed by Steve
	   Willens of Livingston Enterprises for their PortMaster series of
	   Network Access Servers.
	
	11.  Chair's Address
	
	   The RADIUS working group can be contacted via the current chair:
	
	   Carl Rigney
	   Livingston Enterprises
	   4464 Willow Road
	   Pleasanton, California  94588
	
	   Phone: +1 925 737 2100
	   EMail: cdr@telemancy.com
	
	 
	
	 
	
	 
	
	
	Rigney                       Informational                     [Page 26]