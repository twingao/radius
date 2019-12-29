	
	Network Working Group                                           M. Chiba
	Request for Comments: 3576                                    G. Dommety
	Category: Informational                                        M. Eklund
	翻译：twingao                                         Cisco Systems, Inc.
	                                                               D. Mitton
	                                                  Circular Logic, UnLtd.
	                                                                B. Aboba
	                                                   Microsoft Corporation
	                                                               July 2003
	
	
	                     对RADIUS协议的动态认证扩展
	
	备忘录状态
	
	   本文不是在制定一个Internet标准，只是向互联网社区提供相关信息，本文可
	   以不受限制地传播。
	
	
	
	版权说明
	   Copyright (C) The Internet Society (2003).  All Rights Reserved.
	摘要
	   本文描述了对一种对RADIUS协议进行的扩展，该扩展当前已经在部署。该扩展
	   允许动态更改用户会话，像NAS产品已经实现的那样。该扩展包括断开用户连接
	   和更改已经应用于用户会话的授权。
	 
	 
	 
	 
	 
	 
	 
	Chiba, et al.                Informational                      [Page 1]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	目录
	   1.  介绍 . . . . . . . . . . . . . . . . . . . . . . . . . . . . .  3
	       1.1.  Applicability. . . . . . . . . . . . . . . . . . . . . .  3
	       1.2.  Requirements Language  . . . . . . . . . . . . . . . . .  5
	       1.3.  术语 . . . . . . . . . . . . . . . . . . . . . . . . . .  5
	   2.  简介 . . . . . . . . . . . . . . . . . . . . . . . . . . . . .  5
	       2.1.  中断消息（DM） . . . . . . . . . . . . . . . . . . . . .  5
	       2.2.  授权更改消息（CoA）. . . . . . . . . . . . . . . . . . .  6
	       2.3.  报文格式 . . . . . . . . . . . . . . . . . . . . . . . .  7
	   3.  属性 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 11
	       3.1.  Error-Cause. . . . . . . . . . . . . . . . . . . . . . . 13
	       3.2.  属性列表 . . . . . . . . . . . . . . . . . . . . . . . . 16
	   4.  IANA Considerations. . . . . . . . . . . . . . . . . . . . . . 20
	   5.  Security Considerations. . . . . . . . . . . . . . . . . . . . 21
	       5.1.  Authorization Issues . . . . . . . . . . . . . . . . . . 21
	       5.2.  Impersonation. . . . . . . . . . . . . . . . . . . . . . 22
	       5.3.  IPsec Usage Guidelines . . . . . . . . . . . . . . . . . 22
	       5.4.  Replay Protection. . . . . . . . . . . . . . . . . . . . 25
	   6.  Example Traces . . . . . . . . . . . . . . . . . . . . . . . . 26
	   7.  References . . . . . . . . . . . . . . . . . . . . . . . . . . 26
	       7.1.  Normative References . . . . . . . . . . . . . . . . . . 26
	       7.2.  Informative References . . . . . . . . . . . . . . . . . 27
	   8.  Intellectual Property Statement. . . . . . . . . . . . . . . . 28
	   9.  Acknowledgements.  . . . . . . . . . . . . . . . . . . . . . . 28
	   10. Authors' Addresses . . . . . . . . . . . . . . . . . . . . . . 29
	   11. Full Copyright Statement . . . . . . . . . . . . . . . . . . . 30
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	
	Chiba, et al.                Informational                      [Page 2]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	1.  介绍
	   定义在[RFC2865]中的RADIUS协议不支持主动由RADIUS服务器发送往NAS的消
	   息。
	   然而，有很多想要更改已经建立的会话特性的实例，不想使用初始的会话特
	   性。例如：管理员想能够中断正在进行的用户会话。或者，用户想更改授权级
	   别，这需要给用户会话增加或者删除授权属性。
	   为了克服这些限制，某些厂商已经实现了额外的RADIUS命令以能够支持RADIUS
	   服务器主动发送消息给NAS。这些扩展命令提供Disconnect和
	   Change-of-Authorization（CoA）消息。Disconnect消息是用户会话立即中
	   断，而CoA消息修改会话的授权属性，如数据过滤属性。
	1.1.  适应性
	   公开推荐该协议做为Informational RFC而不是standards-track RFC，因为
	   because of problems that cannot be fixed without creating
	   incompatibilities with deployed implementations. 这包括安全弱点，也包
	   括从CoA命令设计的语意不明确。当推荐进行修改时，他们不能强制，因为这会
	   导致和已经存在的实现不兼容。
	   该协议已经存在的实现不支持授权检查。所以一个和其它ISP共享NAS的ISP可能
	   中断或者更改另一个ISP用户的授权。为了补救这个问题，推荐进行“Reverse
	   Path Forwarding”检查。详细情况请参看5.1章节。
	   已存在的实现utilize per-packet authentication and integrity
	   protection algorithms with known weaknesses [MD5Attack].为了提供更强
	   的每报文认证和完整的保护，推荐使用IPSec，详细情况请参见5.3章节。
	 
	 
	 
	Chiba, et al.                Informational                      [Page 3]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   已存在的实现缺乏重放保护，为了支持重发保护，在IPSec重放保护没有实施
	   时，推荐所有的消息中都加上Event-Timestamp属性。实现应该配置成为如果消
	   息中缺少Event-Timestamp属性就静默丢弃。详细情况请参见5.4章节。
	   
	   The approach taken with CoA commands in existing implementations
	   results in a semantic ambiguity.  Existing implementations of the
	   CoA-Request identify the affected session, as well as supply the
	   authorization changes.  Since RADIUS Attributes included within
	   existing implementations of the CoA-Request can be used for session
	   identification or authorization change, it may not be clear which
	   function a given attribute is serving.
	   The problem does not exist within [Diameter], in which authorization
	   change is requested by a command using Attribute Value Pairs (AVPs)
	   solely for identification, resulting in initiation of a standard
	   Request/Response sequence where authorization changes are supplied.
	   As a result, in no command can Diameter AVPs have multiple potential
	   meanings.
	   由于在处理change-of-authorization请求上RADIUS协议和Diameter协议有所不
	   同，
	   Due to differences in handling change-of-authorization requests in
	   RADIUS and Diameter, it may be difficult or impossible for a
	   Diameter/RADIUS gateway to successfully translate existing
	   implementations of this specification to equivalent messages in
	   Diameter.  For example, a Diameter command changing any attribute
	   used for identification within existing CoA-Request implementations
	   cannot be translated, since such an authorization change is
	   impossible to carry out in existing implementations.  Similarly,
	   translation between existing implementations of Disconnect-Request or
	   CoA-Request messages and Diameter is tricky because a Disconnect-
	   Request or CoA-Request message will need to be translated to multiple
	   Diameter commands.
	   To simplify translation between RADIUS and Diameter, a Service-Type
	   Attribute with value "Authorize Only" can (optionally) be included
	   within a Disconnect-Request or CoA-Request.  Such a Request contains
	   only identification attributes.  A NAS supporting the "Authorize
	   Only" Service-Type within a Disconnect-Request or CoA-Request
	   responds with a NAK containing a Service-Type Attribute with value
	   "Authorize Only" and an Error-Cause Attribute with value "Request
	   Initiated".  The NAS will then send an Access-Request containing a
	   Service-Type Attribute with a value of "Authorize Only".  This usage
	   sequence is akin to what occurs in Diameter and so is more easily
	   translated by a Diameter/RADIUS gateway.
	 
	 
	Chiba, et al.                Informational                      [Page 4]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	1.2.  Requirements Language
	   In this document, several words are used to signify the requirements
	   of the specification.  These words are often capitalized.  The key
	   words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
	   "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document
	   are to be interpreted as described in [RFC2119].
	1.3.  Terminology
	   This document frequently uses the following terms:
	   Network Access Server (NAS): The device providing access to the
	                                network.
	   service:                     The NAS provides a service to the user,
	                                such as IEEE 802 or PPP.
	   session:                     Each service provided by the NAS to a
	                                user constitutes a session, with the
	                                beginning of the session defined as the
	                                point where service is first provided
	                                and the end of the session defined as
	                                the point where service is ended.  A
	                                user may have multiple sessions in
	                                parallel or series if the NAS supports
	                                that.
	   silently discard:            This means the implementation discards
	                                the packet without further processing.
	                                The implementation SHOULD provide the
	                                capability of logging the error,
	                                including the contents of the silently
	                                discarded packet, and SHOULD record the
	                                event in a statistics counter.
	2.  概述
	   本节描述了Disconnect和Coa消息最通用的实现特性。
	2.1.  中断消息（DM）
	   为了中断NAS上的用户会话，中断请求报文由RADIUS服务器发送。并丢弃所有关
	   联会话的上下文。中断请求报文发送到UDP端口3799，and identifies the NAS
	   as well as the user session to be terminated by inclusion of the
	   identification attributes described in Section 3.
	 
	Chiba, et al.                Informational                      [Page 5]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   +----------+   Disconnect-Request     +----------+
	   |          |   <--------------------  |          |
	   |    NAS   |                          |  RADIUS  |
	   |          |   Disconnect-Response    |  Server  |
	   |          |   ---------------------> |          |
	   +----------+                          +----------+
	   NAS回应由RADIUS服务器发送的中断请求报文，如果所有的关联会话上下文被丢
	   弃并且用户会话不再连接，NAS回应Disconnect-ACK报文，如果NAS不能中断会
	   话和丢弃所有的关联会话上下文，NAS则发送Disconnect-NAK报文。如果
	   Disconnect-Request报文中包含值为“Authorize Only”的Service-Type属
	   性，则NAS必须（MUST）回应Disconnect-NAK报文，Disconnect-ACK报文必须不
	   能（MUST NOT）发送。如果Disconnect-Request报文中包含不支持的
	   Service-Type属性，则NAS必须（MUST）回应Disconnect-NAK报文，该报文可以
	   （MAY）包含值为“Unsupported Service”的Error-Cause属性。
	   Disconnect-ACK报文可以（MAY）包含值为6（Admin-Reset）的
	   Acct-Terminate-Cause属性（属性类型49，定义在[RFC2866]）。
	2.2.  更改授权消息（CoA）
	   CoA-Request报文包含了动态更改会话授权的信息。通常使用在更改数据过滤
	   上。数据过滤可以是关于入口或者出口的，在额外的标识属性（标识入口或者
	   出口的？？）中发送，在3章节描述。使用的端口和报文格式（在2.3节描述）
	   和Disconnect-Request消息相同。
	   下列属性可以（MAY）在CoA-Request报文中发送：
	   Filter-ID (11) - 表示应用在会话中的数据过滤列表的名称。
	   +----------+      CoA-Request         +----------+
	   |          |  <--------------------   |          |
	   |   NAS    |                          |  RADIUS  |
	   |          |     CoA-Response         |  Server  |
	   |          |   ---------------------> |          |
	   +----------+                          +----------+
	   NAS在回应由RADIUS服务器发送的CoA-Request报文时，如果NAS能够成功地更改
	   用户会话的授权信息。NAS发送CoA-ACK报文。如果请求不成功时发送CoA-NAK报
	   文。如果CoA-Request报文中包含值为“Authorize Only”的Service-Type属
	   性，则NAS必须（MUST）回应CoA-NAK报文，CoA-ACK报文必须不能（MUST NOT）
	   发送。如果CoA-Request报文中包含不支持的Service-Type属性，则NAS必须
	   （MUST）回应CoA-NAK报文，该报文可以（MAY）包含值为“Unsupported
	   Service”的Error-Cause属性。
	  
	
	Chiba, et al.                Informational                      [Page 6]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	 
	2.3.  报文格式
	   Disconnect-Request或者CoA-Request消息使用UDP端口3799做为目的端口，回
	   应时，源端口和目的端口对调，准确地讲，RADIUS报文是封装在UDP报文的数据
	   域中。
	   报文格式如下所示。各个域的数据是从左向右传输的。
	   报文格式由以下域组成：代码域，标识符域，长度域，认证字域和属性域，属
	   性域的格式为类型/长度/值（TLV）。所有的域都和RADIUS [RFC2865]描述的一
	   样。认证字域必须（MUST）和计费请求报文一样的方式计算。
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
	   代码
	      代码域占位1个字节，表示RADIUS报文的类型，接收到无效的代码域必须
	      （MUST）静默丢弃，本次扩展的RADIUS代码（十进制）如下：
	      40 - Disconnect-Request [RFC2882]
	      41 - Disconnect-ACK [RFC2882]
	      42 - Disconnect-NAK [RFC2882]
	      43 - CoA-Request [RFC2882]
	      44 - CoA-ACK [RFC2882]
	      45 - CoA-NAK [RFC2882]
	 
	
	Chiba, et al.                Informational                      [Page 7]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   标识符
	      标识符域占位一个字节，用于匹配请求和回应报文。如果在一个很短的时间
	      内接收到相同的源IP地址、源UDP端口号和相同的标识符域的请求报文，
	      RADIUS服务器就可以认为是重复的请求报文。
	      不像RADIUS协议，Disconnect-Request和CoA-Request消息的重发的责任在
	      RADIUS服务器。如果发送这些消息后，RADIUS服务器没有接收到回应，将重
	      发消息。
	      当属性域的内容发生改变或者是已经收到前一个请求的有效的回应，标识符
	      域必须（MUST）改变。如果仅仅是由于重传而内容没有变化时，标识符必须
	      （MUST）保持不变。
	      如果RADIUS服务器向同一个客户端重发Disconnect-Request和CoA-Request
	      消息，并且属性没有修改，则请求认证字，标识符和源端口必须（MUST）使
	      用原来的值。如果有任何属性发生变化，必须（MUST）使用新的请求认证字
	      和标识符。
	      需要注意的是，如果包含Event-Timestamp属性的话，重发报文时该属性会
	      被修改，属性内容改变时需要一个新的标识符和请求认证字。
	      如果向主代理的请求失败，如果备代理可以获得的话，必须查找（must）备
	      代理，失败算法相关论述在[AAATransport]中描述。既然是一个新的请求，
	      那么必须（MUST）使用新的请求认证字和标识符。当然，如果RADIUS服务器
	      直接发送给客户端，失败机制将没有用处，因为Disconnect和CoA消息需要
	      发送给会话驻留的NAS。
	   长度
	      Length域占位两个字节。它包含了报文中的Code域，Identifier域，
	      Length域，Authenticator域和属性域的总长度。在长度域限定的范围之外
	      的字节必须（MUST）作为填充字节，在接收时不予处理。如果包的实际长度
	      小于长度域中给出的值，该包必须（MUST）被静默丢弃。报文的最小长度是
	      20，最大长度是4096。
	 
	 
	Chiba, et al.                Informational                      [Page 8]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   认证字
	      认证字域占位16个字节。最重要的字节先传输。该域的值用来鉴别客户端和
	      RADIUS计费服务器之间的消息。
	   请求认证字
	      在请求报文中，认证字域是一个占位16个字节的MD5校验码，称为请求认证
	      字。请求认证字计算方法和计费请求报文相同。
	      需要注意的是，由于在Disconnect-Request和CoA-Request报文中没有
	      User-Password属性，Disconnect和CoA-Disconnect的请求认证字不能使用
	      和RADIUS接入请求报文一样计算方法。
	   回应认证字
	      在回应报文（即，Disconnect-ACK，Disconnect-NAK，CoA-ACK或者
	      CoA-NAK）中的认证字称为回应认证字。它包含对一个由Code+Identifier+
	      Length+对应的请求包的请求认证字+回应属性（如果有的话）+共享密钥构
	      成的字节流进行某种方式的MD5哈希计算得出的16个字节的hash值。这个占
	      位16个字节的MD5哈希值被存储在回应报文的Authenticator域中。
	   管理提示：共享密钥（在客户端和RADIUS服务器之间的共享密钥）的长度应该
	   （SHOULD）至少大到不可能猜测的程度。RADIUS服务器必须（MUST）根据
	   RADIUS的UDP报文的源IP地址决定使用那个共享密钥，因此，RADIUS请求才可以
	   被代理。
	   属性
	      在Disconnect和CoA-Request消息中，所有的属性都是强制性的。如果
	      CoA-Request报文包含一个或者多个不支持的属性或者不支持的属性值，则
	      NAS必须（MUST）回应CoA-NAK；如果Disconnect-Request报文包含一个或者
	      多个不支持的属性或者不支持的属性值，则NAS必须（MUST）回应
	      Disconnect-NAK。由CoA-Request报文引起的状态改变必须（MUST）是个原
	      子事务。如果请求成功，发送CoA-ACK报文，此时所有请求的授权更改都必
	      须（MUST）实现。如果CoA-Request没有成功，必须（MUST）发送CoA-NAK，
	      所有请求的授权更改必须都不能（MUST NOT）实现。同样的，如果
	      Disconnect-Request不成功，状态改变必须不能（MUST NOT）发生，并且必
	      须（MUST）发送Disconnect-NAK报文。
	     
	 
	Chiba, et al.                Informational                      [Page 9]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	      包括本规范在内的属性可以（may）用来认证，授权或者其它目的，NAS使用
	      属性做RADIUS认证和计费，这些属性可以（may）不支持包含在
	      Disconnect-Request或者CoA-Request消息中，在属性语意上有不同，
	      This is true even for attributes specified within [RFC2865],
	      [RFC2868], [RFC2869] or [RFC3162] as allowable within
	      Access-Accept messages.
	      既然可能导致无法预测的结果，除了在3.2节中说明的属性，其它属性不应
	      该（SHOULD NOT）包含在Disconnect或者CoA消息中。
	      当使用转发代理时，代理必须（must）能够在报文通过的每个方向上更改报
	      文。当代理转发Disconnect或者CoA-Request报文时，代理可以（MAY）增加
	      Proxy-State属性，当代理转发回应报文时，代理必须（MUST）移除它增加
	      的Proxy-State属性（如果它曾经增加过）。Proxy-State属性总是在其它的
	      Proxy-State属性后增加或者移除，没有其它关于该属性在属性列表中位置
	      的要求。由于是基于整个报文内容鉴别Disconnect和CoA回应报文的，
	      Proxy-State属性的变化会使得原来的签名无效，因此代理需要重新计算，
	      转发代理必须不能（MUST NOT）修改报文中已经存在的Proxy-State，State
	      或者Class属性。
	      如果从服务接收到的Disconnect-Request或者CoA-Request报文中有任何
	      Proxy-State属性，转发代理必须（MUST）在回应服务器时包含这些
	      Proxy-State属性。当转发请求报文时，转发代理可以（MAY）在
	      Disconnect-Request或者CoA-Request报文中包含Proxy-State属性，也可以
	      （MAY）在转发时略去这些属性，如果转发代理略去请求报文中的
	      Proxy-State属性，必须（MUST）在发送回服务器之前将它们重新附加到回
	      应报文中。
	 
	 
	 
	 
	Chiba, et al.                Informational                     [Page 10]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	3.  属性
	   在Disconnect-Request和CoA-Request报文中，某些属性用来唯一标识NAS和NAS
	   中用户的会话。为了Disconnect-Request或者CoA-Request成功处理，所有包含
	   在请求消息中的标识属性必须（MUST）匹配，否则应该（SHOULD）发送
	   Disconnect-NAK或者CoA-NAK报文。如果存在的话，会话标识属性：User-Name
	   和Acct-Session-Id属性必须（MUST）匹配，其它的会话标识属性也应该
	   （SHOULD）匹配。当有检测到一个会话标识属性错误，应该（SHOULD）发送
	   Disconnect-NAK或者CoA-NAK报文。使用NAS或者会话标识属性匹配唯一或者多
	   个会话的讨论超出了本文的范围。标识属性包括NAS和会话标识属性，如下表所示：
	   NAS标识属性
	   Attribute             #    Reference  Description
	   ---------            ---   ---------  -----------
	   NAS-IP-Address        4    [RFC2865]  The IPv4 address of the NAS.
	   NAS-Identifier       32    [RFC2865]  String identifying the NAS.
	   NAS-IPv6-Address     95    [RFC3162]  The IPv6 address of the NAS.
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	 
	Chiba, et al.                Informational                     [Page 11]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   会话标识属性
	   Attribute              #    Reference  Description
	   ---------             ---   ---------  -----------
	   User-Name              1    [RFC2865]  The name of the user
	                                          associated with the session.
	   NAS-Port               5    [RFC2865]  The port on which the
	                                          session is terminated.
	   Framed-IP-Address      8    [RFC2865]  The IPv4 address associated
	                                          with the session.
	   Called-Station-Id     30    [RFC2865]  The link address to which
	                                          the session is connected.
	   Calling-Station-Id    31    [RFC2865]  The link address from which
	                                          the session is connected.
	   Acct-Session-Id       44    [RFC2866]  The identifier uniquely
	                                          identifying the session
	                                          on the NAS.
	   Acct-Multi-Session-Id 50    [RFC2866]  The identifier uniquely
	                                          identifying related sessions.
	   NAS-Port-Type         61    [RFC2865]  The type of port used.
	   NAS-Port-Id           87    [RFC2869]  String identifying the port
	                                          where the session is.
	   Originating-Line-Info 94    [NASREQ]   Provides information on the
	                                          characteristics of the line
	                                          from which a session
	                                          originated.
	   Framed-Interface-Id   96    [RFC3162]  The IPv6 Interface Identifier
	                                          associated with the session;
	                                          always sent with
	                                          Framed-IPv6-Prefix.
	   Framed-IPv6-Prefix    97    [RFC3162]  The IPv6 prefix associated
	                                          with the session, always sent
	                                          with Framed-Interface-Id.
	   从5.1节的安全考虑，User-Name属性应该包（SHOULD）含在
	   Disconnect-Request或CoA-Request报文中；也可以（MAY）包含一个或者多个
	   其它的会话标识属性。从5.2节的安全考虑，NAS-IP-Address或者
	   NAS-IPv6-Address属性应该（SHOULD）包含在Disconnect-Request或
	   CoA-Request报文中；也可以（MAY）包含NAS-Identifier属性。
	   如果在CoA-Request报文中指明的一个或者多个授权更改不能实施，或者一个或
	   者多个属性或者属性值不支持的话，必须（MUST）发送CoA-NAK报文。同样地，
	   如果Disconnect-Request报文中有一个或者多个不支持的属性或者属性值，那
	   么必须（MUST）发送Disconnect-NAK报文。
	 
	
	Chiba, et al.                Informational                     [Page 12]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	   当值为“Authorize Only”的Service-Type属性包含在CoA-Request或
	   Disconnect-Request报文中时，必须不能包含（MUST NOT）表示授权更改的属
	   性，只允许包含标识属性。如果在CoA-Request报文没有包含NAS或者会话标识
	   属性，必须（MUST）发送CoA-NAK报文；可以（MAY）包含一个值为
	   “Unsupported Attribute”的Error-Cause属性。同样地，如果在
	   Disconnect-Request报文没有包含NAS或者会话标识属性，必须（MUST）发送
	   Disconnect-NAK报文；可以（MAY）包含一个值为“Unsupported Attribute”
	   的Error-Cause属性。
	3.1.  Error-Cause
	   描述
	      可能某种原因，NAS不能实现Disconnect-Request或CoA-Request消息，
	      Error-Cause属性提供了该问题的详细原因。它可以（MAY）包含在
	      Disconnect-ACK，Disconnect-NAK和CoA-NAK消息中。
	      Error-Cause属性的格式如下所示。各个域是按照自左向右的顺序传输的。
	    0                   1                   2                   3
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |     Type      |    Length     |             Value
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	              Value (cont)         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   类型
	      101 代表Error-Cause
	   长度
	      6
	   值
	      值域占位4个字节，包含了表示错误原因的整数。值0-199和300-399保留，
	      值200-299表示成功完成，因此这些值只可能出现在Disconnect-ACK或者
	      CoA-ACK消息中，一定不能（MUST NOT）在Disconnect-NAK或CoA-NAK消息中
	      发送。值400-499表示由RADIUS服务器引起的致命错误，因此这些值可以
	      （MAY）出现在CoA-NAK或Disconnect-NAK消息中，必须不能（MUST NOT）出
	      现在CoA-ACK或Disconnect-ACK消息中，值500-599表示发生NAS或者RADIUS
	      代理上的致命错误，因此它们可以（MAY）在CoA-NAK和Disconnect-NAK消息
	      中发送，必须不能（MUST NOT）在CoA-ACK或Disconnect-ACK消息中发送。
	      Error-Cause属性值应该（SHOULD）有RADIUS记录下来，Error-Code属性值
	      （十进制）包括如下：
	     
	
	Chiba, et al.                Informational                     [Page 13]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	 
	    #     Value
	   ---    -----
	   201    Residual Session Context Removed（残留的会话上下文已经被删除）
	   202    Invalid EAP Packet (Ignored)（无效的EAP报文）
	   401    Unsupported Attribute（不支持的属性）
	   402    Missing Attribute（丢失属性）
	   403    NAS Identification Mismatch（NAS标识不匹配）
	   404    Invalid Request（无效的请求）
	   405    Unsupported Service（不支持的业务）
	   406    Unsupported Extension（不支持扩展）
	   501    Administratively Prohibited（管理禁止）
	   502    Request Not Routable (Proxy)（请求不可路由）
	   503    Session Context Not Found（会话上下文不能找到）
	   504    Session Context Not Removable（会话上下文不能移除）
	   505    Other Proxy Processing Error（其它代理处理错误）
	   506    Resources Unavailable（资源不可获得）
	   507    Request Initiated（请求？？）
	   值“会话上下文已经被删除”在Disconnect-Request报文的回应报文中发送，
	   如果用户会话不再活动，但残留的会话上下文被发现并且成功移除，该值只能
	   在Disconnect-ACK报文中发送，必须不能（MUST NOT）在CoA-ACK，
	   Disconnect-NAK或CoA-NAK报文中发送。
	   值“无效的EAP报文”是一个非致命错误，被协议的实现必须不能（MUST NOT）
	   发送该值。
	   值“不支持的属性”是一个致命错误，如果请求报文包含了一个不支持的属性
	   （如Vendor-Specific或EAP-Message属性）。
	   
	   值“丢失属性”是一个致命错误，如果请求报文中丢失了重要的属性（如NAS或
	   者会话标识属性）。
	   值“NAS标识不匹配”是一个致命错误，如果接收到的一个或者多个NAS标识属性（参看
	   第3节）和NAS的标识不匹配。
	 
	
	Chiba, et al.                Informational                     [Page 14]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   值“无效的请求”是一个致命错误，如果请求的其它方面无效的话，如果一个
	   或者多个属性（如EAP-Message属性）格式不正确。
	   值“不支持的业务”是一个致命错误，如果请求报文中的Service-Type属性发
	   送了无效的或者不支持的值。
	   值“不支持扩展”是个致命的错误，由于缺少对扩展的支持，如Disconnect
	   和/或CoA消息，试图转发请求报文给NAS时，代理接收到ICMP端口不可达消息，
	   通常会发送该值。
	   值“管理禁止”是个致命错误，如果NAS被配置禁止处理对某个特定会话的请求
	   消息。
	   值“请求不可路由”是个致命的错误，它可以（MAY）由RADIUS代理发送，但必
	   须不能（MUST NOT）由NAS发送。表示RADIUS代理不能决定如何路由请求报文给
	   NAS。例如：如果需要的表项不在代理的域路由表中。
	   值“会话上下文不能找到”是个致命错误，如果请求报文中表示的会话上下文
	   在NAS不存在。
	   "Session Context Not Removable" is a fatal error sent in response to
	   a Disconnect-Request if the NAS was able to locate the session
	   context, but could not remove it for some reason.  It MUST NOT be
	   sent within a CoA-ACK, CoA-NAK or Disconnect-ACK, only within a
	   Disconnect-NAK.
	   "Other Proxy Processing Error" is a fatal error sent in response to a
	   Request that could not be processed by a proxy, for reasons other
	   than routing.
	   "Resources Unavailable" is a fatal error sent when a Request could
	   not be honored due to lack of available NAS resources (memory, non-
	   volatile storage, etc.).
	   "Request Initiated" is a fatal error sent in response to a Request
	   including a Service-Type Attribute with a value of "Authorize Only".
	   It indicates that the Disconnect-Request or CoA-Request has not been
	   honored, but that a RADIUS Access-Request including a Service-Type
	   Attribute with value "Authorize Only" is being sent to the RADIUS
	   server.
	 
	 
	Chiba, et al.                Informational                     [Page 15]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	3.2.  属性列表
	   下表列出了哪个属性可以出现在哪种类型的报文中，以及可以出现几次：
	   Change-of-Authorization消息
	   Request   ACK      NAK   #   Attribute
	   0-1       0        0     1   User-Name [Note 1]
	   0-1       0        0     4   NAS-IP-Address [Note 1]
	   0-1       0        0     5   NAS-Port [Note 1]
	   0-1       0        0-1   6   Service-Type [Note 6]
	   0-1       0        0     7   Framed-Protocol [Note 3]
	   0-1       0        0     8   Framed-IP-Address [Note 1]
	   0-1       0        0     9   Framed-IP-Netmask [Note 3]
	   0-1       0        0    10   Framed-Routing [Note 3]
	   0+        0        0    11   Filter-ID [Note 3]
	   0-1       0        0    12   Framed-MTU [Note 3]
	   0+        0        0    13   Framed-Compression [Note 3]
	   0+        0        0    14   Login-IP-Host [Note 3]
	   0-1       0        0    15   Login-Service [Note 3]
	   0-1       0        0    16   Login-TCP-Port [Note 3]
	   0+        0        0    18   Reply-Message [Note 2]
	   0-1       0        0    19   Callback-Number [Note 3]
	   0-1       0        0    20   Callback-Id [Note 3]
	   0+        0        0    22   Framed-Route [Note 3]
	   0-1       0        0    23   Framed-IPX-Network [Note 3]
	   0-1       0-1      0-1  24   State [Note 7]
	   0+        0        0    25   Class [Note 3]
	   0+        0        0    26   Vendor-Specific [Note 3]
	   0-1       0        0    27   Session-Timeout [Note 3]
	   0-1       0        0    28   Idle-Timeout [Note 3]
	   0-1       0        0    29   Termination-Action [Note 3]
	   0-1       0        0    30   Called-Station-Id [Note 1]
	   0-1       0        0    31   Calling-Station-Id [Note 1]
	   0-1       0        0    32   NAS-Identifier [Note 1]
	   0+        0+       0+   33   Proxy-State
	   0-1       0        0    34   Login-LAT-Service [Note 3]
	   0-1       0        0    35   Login-LAT-Node [Note 3]
	   0-1       0        0    36   Login-LAT-Group [Note 3]
	   0-1       0        0    37   Framed-AppleTalk-Link [Note 3]
	   0+        0        0    38   Framed-AppleTalk-Network [Note 3]
	   0-1       0        0    39   Framed-AppleTalk-Zone [Note 3]
	   0-1       0        0    44   Acct-Session-Id [Note 1]
	   0-1       0        0    50   Acct-Multi-Session-Id [Note 1]
	   0-1       0-1      0-1  55   Event-Timestamp
	   0-1       0        0    61   NAS-Port-Type [Note 1]
	   Request   ACK      NAK   #   Attribute
	 
	Chiba, et al.                Informational                     [Page 16]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   Request   ACK      NAK   #   Attribute
	   0-1       0        0    62   Port-Limit [Note 3]
	   0-1       0        0    63   Login-LAT-Port [Note 3]
	   0+        0        0    64   Tunnel-Type [Note 5]
	   0+        0        0    65   Tunnel-Medium-Type [Note 5]
	   0+        0        0    66   Tunnel-Client-Endpoint [Note 5]
	   0+        0        0    67   Tunnel-Server-Endpoint [Note 5]
	   0+        0        0    69   Tunnel-Password [Note 5]
	   0-1       0        0    71   ARAP-Features [Note 3]
	   0-1       0        0    72   ARAP-Zone-Access [Note 3]
	   0+        0        0    78   Configuration-Token [Note 3]
	   0+        0-1      0    79   EAP-Message [Note 2]
	   0-1       0-1      0-1  80   Message-Authenticator
	   0+        0        0    81   Tunnel-Private-Group-ID [Note 5]
	   0+        0        0    82   Tunnel-Assignment-ID [Note 5]
	   0+        0        0    83   Tunnel-Preference [Note 5]
	   0-1       0        0    85   Acct-Interim-Interval [Note 3]
	   0-1       0        0    87   NAS-Port-Id [Note 1]
	   0-1       0        0    88   Framed-Pool [Note 3]
	   0+        0        0    90   Tunnel-Client-Auth-ID [Note 5]
	   0+        0        0    91   Tunnel-Server-Auth-ID [Note 5]
	   0-1       0        0    94   Originating-Line-Info [Note 1]
	   0-1       0        0    95   NAS-IPv6-Address [Note 1]
	   0-1       0        0    96   Framed-Interface-Id [Note 1]
	   0+        0        0    97   Framed-IPv6-Prefix [Note 1]
	   0+        0        0    98   Login-IPv6-Host [Note 3]
	   0+        0        0    99   Framed-IPv6-Route [Note 3]
	   0-1       0        0   100   Framed-IPv6-Pool [Note 3]
	   0         0        0+  101   Error-Cause
	   Request   ACK      NAK   #   Attribute
	   Disconnect消息
	   Request   ACK      NAK   #   Attribute
	   0-1       0        0     1   User-Name [Note 1]
	   0-1       0        0     4   NAS-IP-Address [Note 1]
	   0-1       0        0     5   NAS-Port [Note 1]
	   0-1       0        0-1   6   Service-Type [Note 6]
	   0-1       0        0     8   Framed-IP-Address [Note 1]
	   0+        0        0    18   Reply-Message [Note 2]
	   0-1       0-1      0-1  24   State [Note 7]
	   0+        0        0    25   Class [Note 4]
	   0+        0        0    26   Vendor-Specific
	   0-1       0        0    30   Called-Station-Id [Note 1]
	   0-1       0        0    31   Calling-Station-Id [Note 1]
	   0-1       0        0    32   NAS-Identifier [Note 1]
	   0+        0+       0+   33   Proxy-State
	   Request   ACK      NAK   #   Attribute
	 
	Chiba, et al.                Informational                     [Page 17]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   Request   ACK      NAK   #   Attribute
	   0-1       0        0    44   Acct-Session-Id [Note 1]
	   0-1       0-1      0    49   Acct-Terminate-Cause
	   0-1       0        0    50   Acct-Multi-Session-Id [Note 1]
	   0-1       0-1      0-1  55   Event-Timestamp
	   0-1       0        0    61   NAS-Port-Type [Note 1]
	   0+        0-1      0    79   EAP-Message [Note 2]
	   0-1       0-1      0-1  80   Message-Authenticator
	   0-1       0        0    87   NAS-Port-Id [Note 1]
	   0-1       0        0    94   Originating-Line-Info [Note 1]
	   0-1       0        0    95   NAS-IPv6-Address [Note 1]
	   0-1       0        0    96   Framed-Interface-Id [Note 1]
	   0+        0        0    97   Framed-IPv6-Prefix [Note 1]
	   0         0+       0+  101   Error-Cause
	   Request   ACK      NAK   #   Attribute
	   [注1]当NAS或会话标识属性包含在Disconnect-Request或CoA-Request消息中
	   时，它们只做为标识之用。这些属性必须不能（MUST NOT）用于标识之外的目
	   的，（即在CoA-Request消息中请求授权更改）。
	   [注2]Reply-Message属性被用来表示显示给用户的消息，该消息只在
	   Disconnect-Request或CoA-Request报文成功处理时（后续会发送
	   Disconnect-ACK或CoA-ACK报文）显示。当用EAP做为认证方式，
	   EAP-Message/Notification-Request属性做为显示给用户的消息，
	   Disconnect-ACK或CoA-ACK消息包含EAP-Message/Notification-Response属
	   性。
	   [注3]当包含在CoA-Request报文中时，这些属性表示授权更改请求，当这些属
	   性之一在CoA-Request报文中略去，NAS假定该属性值保持不变，包含在
	   CoA-Request报文中的属性替换了所有的已存在的相同属性类型的值。
	   [注4]当包含在成功处理的Disconnect-Request中时（后续会发送
	   Disconnect-ACK报文），Class属性应该（SHOULD）在由客户端发送往计费服务
	   器的计费结束报文中不能被修改。如果Disconnect-Request报文没有成功处
	   理，那么Class属性不能被处理。
	   [注5]当包含在CoA-Request报文中时，这些属性表示授权更改请求，当在成功
	   处理的CoA-Request报文中发送隧道属性，删除所有已存在的隧道属性，以修改
	   属性代替。
	 
	 
	Chiba, et al.                Informational                     [Page 18]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   [注6]当包含在Disconnect-Request或CoA-Request消息中时，一个值为
	   “Authorize Only”的Service-Type属性表示请求报文只包含了NAS和会话表示
	   属性，NAS应该（should）试图通过发送携带值为“Authorize Only”的
	   Service-Type属性的接入请求报文重认证。这种用法和Diameter协议的用法类
	   似，因此很容易在这两个协议中传输。在CoA-Request和Disconnect-Request消
	   息中支持Service-Type属性是可选的，当没有包含Service-Type属性时，请求
	   报文可能（may）包含标识和授权属性。如果NAS不支持在Disconnect-Request
	   消息中包含值为“Authorize Only”的Service-Type属性，则必须（MUST）回
	   应一个不携带Service-Type属性的Disconnect-NAK消息，该消息可能（MAY）包
	   含值为“Unsupported Service”的Error-Cause属性。如果NAS不支持在
	   CoA-Request消息中包含值为“Authorize Only”的Service-Type属性，则必须
	   （MUST）回应一个不携带Service-Type属性的CoA-NAK消息，该消息可能（MAY）包
	   含值为“Unsupported Service”的Error-Cause属性。
	   支持在Disconnect-Request或CoA-Request消息中携带值为“Authorize Only”
	   的Service-Type属性的NAS必须（MUST）分别回应Disconnect-NAK或CoA-NAK消
	   息，该消息包含值为“Authorize Only”的Service-Type属性和值为“Request
	   Initiated”的Error-Cause属性。NAS然后发送一个携带值为“Authorize
	   Only”的Service-Type属性的接入请求报文给RADIUS服务器。该接入请求报文
	   应该（SHOULD）包含Disconnect或CoA-Request的NAS属性，如可以合法的包含
	   在接入请求报文（在[RFC2865]，[RFC2868]，[RFC2869]和[RFC3162]定义）中
	   的会话属性。如[RFC2869]的第5.19节提到的，Message-Authenticator属性应
	   该（SHOULD）包含在没有包含User-Password，CHAP-Password，ARAP-Password
	   或EAP-Message属性的接入请求报文中，RADIUS服务器应该（should）发送回接
	   入成功回应报文（重新）授权会话或者发送回接入拒绝回应报文拒绝（重新）
	   授权会话。
	   [注7]State属性用在由RADIUS服务器发送往NAS的Disconnect-Request或
	   CoA-Request消息中，必须（MUST）不能更改地在后续ACK或AK消息中由NAS发送
	   回服务器。如果携带了State属性的Disconnect-Request或CoA-Request消息中
	   携带了值为“Authorize Only”的Service-Type属性。那么如果有State属性的
	   话，在后续发往RADIUS服务器的接入请求报文中必须（MUST）携带上没有更改
	   的State属性。State属性也用在由RADIUS服务器发往NAS的携带了值为
	   “RADIUS-Request”的Termination-Action属性的CoA-Request报文中。如果当
	   前会话中断时，客户端根据Termination-Action属性发送了一个新的接入请求
	   报文，该报文必须（MUST）携带上没有更改的State属性，在以上的任一个用法
	   中，客户端必须不能（MUST NOT）在本地解释State属性。Disconnect-Request
	   或CoA-Request报文必须（must）只能有0个或者1个State属性。State属性的用
	   法由各实现独立实现。如果RADIUS服务在接入请求报文中不认可State属性，它
	   必须（MUST）发送接入拒绝回应报文。
	 
	Chiba, et al.                Informational                     [Page 19]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   下表说明上表各表项的含义：
	   0     该属性必须不能（MUST NOT）出现在该类型报文中。
	   0+    0个或者多个该属性的实例可以（MAY）出现在该类型报文中。
	   0-1   0个或者1个该属性的实例可以（MAY）出现在该类型报文中。
	   1     该属性必须且只能有（MUST）一个实例出现在该类型的报文中。
	
	4.  IANA事项
	   本文档使用RADIUS [RFC2865]命名空间，参见
	   <http://www.iana.org/assignments/radius-types>。本节有六个需要修改的
	   地方：RADIUS报文类型代码。这些报文类型分配在[RADIANA]中：
	   40 - Disconnect-Request
	   41 - Disconnect-ACK
	   42 - Disconnect-NAK
	   43 - CoA-Request
	   44 - CoA-ACK
	   45 - CoA-NAK
	   分配了一个新的Service-Type属性值“Authorize Only”。本文档也使用了UDP
	   [RFC768]名称空间，参见<http://www.iana.org/assignments/port-numbers>。
	   作者要求从注册端口号范围内分配一个端口号。最后本文给Error-Cause属性分
	   配了如下的十进制值：
	    #     Value
	   ---    -----
	   201    Residual Session Context Removed
	   202    Invalid EAP Packet (Ignored)
	   401    Unsupported Attribute
	   402    Missing Attribute
	   403    NAS Identification Mismatch
	   404    Invalid Request
	   405    Unsupported Service
	   406    Unsupported Extension
	   501    Administratively Prohibited
	   502    Request Not Routable (Proxy)
	 
	Chiba, et al.                Informational                     [Page 20]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   503    Session Context Not Found
	   504    Session Context Not Removable
	   505    Other Proxy Processing Error
	   506    Resources Unavailable
	   507    Request Initiated
	5.  Security Considerations
	5.1.  Authorization Issues
	   Where a NAS is shared by multiple providers, it is undesirable for
	   one provider to be able to send Disconnect-Request or CoA-Requests
	   affecting the sessions of another provider.
	   A NAS or RADIUS proxy MUST silently discard Disconnect-Request or
	   CoA-Request messages from untrusted sources.  By default, a RADIUS
	   proxy SHOULD perform a "reverse path forwarding" (RPF) check to
	   verify that a Disconnect-Request or CoA-Request originates from an
	   authorized RADIUS server.  In addition, it SHOULD be possible to
	   explicitly authorize additional sources of Disconnect-Request or
	   CoA-Request packets relating to certain classes of sessions.  For
	   example, a particular source can be explicitly authorized to send
	   CoA-Request messages relating to users within a set of realms.
	   To perform the RPF check, the proxy uses the session identification
	   attributes included in Disconnect-Request or CoA-Request messages, in
	   order to determine the RADIUS server(s) to which an equivalent
	   Access-Request could be routed.  If the source address of the
	   Disconnect-Request or CoA-Request is within this set, then the
	   Request is forwarded; otherwise it MUST be silently discarded.
	   Typically the proxy will extract the realm from the Network Access
	   Identifier [RFC2486] included within the User-Name Attribute, and
	   determine the corresponding RADIUS servers in the proxy routing
	   tables.  The RADIUS servers for that realm  are then compared against
	   the source address of the packet.  Where no RADIUS proxy is present,
	   the RPF check will need to be performed by the NAS itself.
	   Since authorization to send a Disconnect-Request or CoA-Request is
	   determined based on the source address and the corresponding shared
	   secret, the NASes or proxies SHOULD configure a different shared
	   secret for each RADIUS server.
	 
	 
	 
	 
	Chiba, et al.                Informational                     [Page 21]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	5.2.  Impersonation
	   [RFC2865] Section 3 states:
	      A RADIUS server MUST use the source IP address of the RADIUS UDP
	      packet to decide which shared secret to use, so that RADIUS
	      requests can be proxied.
	   When RADIUS requests are forwarded by a proxy, the NAS-IP-Address or
	   NAS-IPv6-Address Attributes will typically not match the source
	   address observed by the RADIUS server.  Since the NAS-Identifier
	   Attribute need not contain an FQDN, this attribute may not be
	   resolvable to the source address observed by the RADIUS server, even
	   when no proxy is present.
	   As a result, the authenticity check performed by a RADIUS server or
	   proxy does not verify the correctness of NAS identification
	   attributes.  This makes it possible for a rogue NAS to forge NAS-IP-
	   Address, NAS-IPv6-Address or NAS-Identifier Attributes within a
	   RADIUS Access-Request in order to impersonate another NAS.  It is
	   also possible for a rogue NAS to forge session identification
	   attributes such as the Called-Station-Id, Calling-Station-Id, or
	   Originating-Line-Info [NASREQ].  This could fool the RADIUS server
	   into sending Disconnect-Request or CoA-Request messages containing
	   forged session identification attributes to a NAS targeted by an
	   attacker.
	   To address these vulnerabilities RADIUS proxies SHOULD check whether
	   NAS identification attributes (see Section 3.) match the source
	   address of packets originating from the NAS.  Where one or more
	   attributes do not match, Disconnect-Request or CoA-Request messages
	   SHOULD be silently discarded.
	   Such a check may not always be possible.  Since the NAS-Identifier
	   Attribute need not correspond to an FQDN, it may not be resolvable to
	   an IP address to be matched against the source address.  Also, where
	   a NAT exists between the RADIUS client and proxy, checking the NAS-
	   IP-Address or NAS-IPv6-Address Attributes may not be feasible.
	5.3.  IPsec Usage Guidelines
	   In addition to security vulnerabilities unique to Disconnect or CoA
	   messages, the protocol exchanges described in this document are
	   susceptible to the same vulnerabilities as RADIUS [RFC2865].  It is
	   RECOMMENDED that IPsec be employed to afford better security.
	 
	 
	
	Chiba, et al.                Informational                     [Page 22]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   Implementations of this specification SHOULD support IPsec [RFC2401]
	   along with IKE [RFC2409] for key management.  IPsec ESP [RFC2406]
	   with a non-null transform SHOULD be supported, and IPsec ESP with a
	   non-null encryption transform and authentication support SHOULD be
	   used to provide per-packet confidentiality, authentication, integrity
	   and replay protection.  IKE SHOULD be used for key management.
	   Within RADIUS [RFC2865], a shared secret is used for hiding
	   Attributes such as User-Password, as well as used in computation of
	   the Response Authenticator.  In RADIUS accounting [RFC2866], the
	   shared secret is used in computation of both the Request
	   Authenticator and the Response Authenticator.
	   Since in RADIUS a shared secret is used to provide confidentiality as
	   well as integrity protection and authentication, only use of IPsec
	   ESP with a non-null transform can provide security services
	   sufficient to substitute for RADIUS application-layer security.
	   Therefore, where IPsec AH or ESP null is used, it will typically
	   still be necessary to configure a RADIUS shared secret.
	   Where RADIUS is run over IPsec ESP with a non-null transform, the
	   secret shared between the NAS and the RADIUS server MAY NOT be
	   configured.  In this case, a shared secret of zero length MUST be
	   assumed.  However, a RADIUS server that cannot know whether incoming
	   traffic is IPsec-protected MUST be configured with a non-null RADIUS
	   shared secret.
	   When IPsec ESP is used with RADIUS, per-packet authentication,
	   integrity and replay protection MUST be used.  3DES-CBC MUST be
	   supported as an encryption transform and AES-CBC SHOULD be supported.
	   AES-CBC SHOULD be offered as a preferred encryption transform if
	   supported.  HMAC-SHA1-96 MUST be supported as an authentication
	   transform.  DES-CBC SHOULD NOT be used as the encryption transform.
	   A typical IPsec policy for an IPsec-capable RADIUS client is
	   "Initiate IPsec, from me to any destination port UDP 1812".  This
	   IPsec policy causes an IPsec SA to be set up by the RADIUS client
	   prior to sending RADIUS traffic.  If some RADIUS servers contacted by
	   the client do not support IPsec, then a more granular policy will be
	   required: "Initiate IPsec, from me to IPsec-Capable-RADIUS-Server,
	   destination port UDP 1812."
	   For a client implementing this specification, the policy would be
	   "Accept IPsec, from any to me, destination port UDP 3799".  This
	   causes the RADIUS client to accept (but not require) use of IPsec.
	   It may not be appropriate to require IPsec for all RADIUS servers
	   connecting to an IPsec-enabled RADIUS client, since some RADIUS
	   servers may not support IPsec.
	 
	Chiba, et al.                Informational                     [Page 23]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   For an IPsec-capable RADIUS server, a typical IPsec policy is "Accept
	   IPsec, from any to me, destination port 1812".  This causes the
	   RADIUS server to accept (but not require) use of IPsec.  It may not
	   be appropriate to require IPsec for all RADIUS clients connecting to
	   an IPsec-enabled RADIUS server, since some RADIUS clients may not
	   support IPsec.
	   For servers implementing this specification, the policy would be
	   "Initiate IPsec, from me to any, destination port UDP 3799".  This
	   causes the RADIUS server to initiate IPsec when sending RADIUS
	   extension traffic to any RADIUS client.  If some RADIUS clients
	   contacted by the server do not support IPsec, then a more granular
	   policy will be required, such as "Initiate IPsec, from me to IPsec-
	   capable-RADIUS-client, destination port UDP 3799".
	   Where IPsec is used for security, and no RADIUS shared secret is
	   configured, it is important that the RADIUS client and server perform
	   an authorization check.  Before enabling a host to act as a RADIUS
	   client, the RADIUS server SHOULD check whether the host is authorized
	   to provide network access.  Similarly, before enabling a host to act
	   as a RADIUS server, the RADIUS client SHOULD check whether the host
	   is authorized for that role.
	   RADIUS servers can be configured with the IP addresses (for IKE
	   Aggressive Mode with pre-shared keys) or FQDNs (for certificate
	   authentication) of RADIUS clients.  Alternatively, if a separate
	   Certification Authority (CA) exists for RADIUS clients, then the
	   RADIUS server can configure this CA as a trust anchor [RFC3280] for
	   use with IPsec.
	   Similarly, RADIUS clients can be configured with the IP addresses
	   (for IKE Aggressive Mode with pre-shared keys) or FQDNs (for
	   certificate authentication) of RADIUS servers.  Alternatively, if a
	   separate CA exists for RADIUS servers, then the RADIUS client can
	   configure this CA as a trust anchor for use with IPsec.
	   Since unlike SSL/TLS, IKE does not permit certificate policies to be
	   set on a per-port basis, certificate policies need to apply to all
	   uses of IPsec on RADIUS clients and servers.  In IPsec deployment
	   supporting only certificate authentication, a management station
	   initiating an IPsec-protected telnet session to the RADIUS server
	   would need to obtain a certificate chaining to the RADIUS client CA.
	   Issuing such a certificate might not be appropriate if the management
	   station was not authorized as a RADIUS client.
	   Where RADIUS clients may obtain their IP address dynamically (such as
	   an Access Point supporting DHCP), Main Mode with pre-shared keys
	   [RFC2409] SHOULD NOT be used, since this requires use of a group
	 
	Chiba, et al.                Informational                     [Page 24]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   pre-shared key; instead, Aggressive Mode SHOULD be used.  Where
	   RADIUS client addresses are statically assigned, either Aggressive
	   Mode or Main Mode MAY be used.  With certificate authentication, Main
	   Mode SHOULD be used.
	   Care needs to be taken with IKE Phase 1 Identity Payload selection in
	   order to enable mapping of identities to pre-shared keys, even with
	   Aggressive Mode.  Where the ID_IPV4_ADDR or ID_IPV6_ADDR Identity
	   Payloads are used and addresses are dynamically assigned, mapping of
	   identities to keys is not possible, so that group pre-shared keys are
	   still a practical necessity.  As a result, the ID_FQDN identity
	   payload SHOULD be employed in situations where Aggressive mode is
	   utilized along with pre-shared keys and IP addresses are dynamically
	   assigned.  This approach also has other advantages, since it allows
	   the RADIUS server and client to configure themselves based on the
	   fully qualified domain name of their peers.
	   Note that with IPsec, security services are negotiated at the
	   granularity of an IPsec SA, so that RADIUS exchanges requiring a set
	   of security services different from those negotiated with existing
	   IPsec SAs will need to negotiate a new IPsec SA.  Separate IPsec SAs
	   are also advisable where quality of service considerations dictate
	   different handling RADIUS conversations.  Attempting to apply
	   different quality of service to connections handled by the same IPsec
	   SA can result in reordering, and falling outside the replay window.
	   For a discussion of the issues, see [RFC2983].
	5.4.  Replay Protection
	   Where IPsec replay protection is not used, the Event-Timestamp (55)
	   Attribute [RFC2869] SHOULD be included within all messages.  When
	   this attribute is present, both the NAS and the RADIUS server MUST
	   check that the Event-Timestamp Attribute is current within an
	   acceptable time window.  If the Event-Timestamp Attribute is not
	   current, then the message MUST be silently discarded.  This implies
	   the need for time synchronization within the network, which can be
	   achieved by a variety of means, including secure NTP, as described in
	   [NTPAUTH].
	   Both the NAS and the RADIUS server SHOULD be configurable to silently
	   discard messages lacking an Event-Timestamp Attribute.  A default
	   time window of 300 seconds is recommended.
	 
	 
	 
	 
	Chiba, et al.                Informational                     [Page 25]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	6.  Example Traces
	   Disconnect Request with User-Name:
	    0: xxxx xxxx xxxx xxxx xxxx 2801 001c 1b23    .B.....$.-(....#
	   16: 624c 3543 ceba 55f1 be55 a714 ca5e 0108    bL5C..U..U...^..
	   32: 6d63 6869 6261
	   Disconnect Request with Acct-Session-ID:
	    0: xxxx xxxx xxxx xxxx xxxx 2801 001e ad0d    .B..... ~.(.....
	   16: 8e53 55b6 bd02 a0cb ace6 4e38 77bd 2c0a    .SU.......N8w.,.
	   32: 3930 3233 3435 3637                        90234567
	   Disconnect Request with Framed-IP-Address:
	    0: xxxx xxxx xxxx xxxx xxxx 2801 001a 0bda    .B....."2.(.....
	   16: 33fe 765b 05f0 fd9c c32a 2f6b 5182 0806    3.v[.....*/kQ...
	   32: 0a00 0203
	7.  References
	7.1.  Normative References
	   [RFC1305]      Mills, D., "Network Time Protocol (version 3)
	                  Specification, Implementation and Analysis", RFC 1305,
	                  March 1992.
	   [RFC1321]      Rivest, R., "The MD5 Message-Digest Algorithm", RFC
	                  1321, April 1992.
	   [RFC2104]      Krawczyk, H., Bellare, M. and R. Canetti, "HMAC:
	                  Keyed-Hashing for Message Authentication", RFC 2104,
	                  February 1997.
	   [RFC2119]      Bradner, S., "Key words for use in RFCs to Indicate
	                  Requirement Levels", BCP 14, RFC 2119, March 1997.
	   [RFC2401]      Kent, S. and R. Atkinson, "Security Architecture for
	                  the Internet Protocol", RFC 2401, November 1998.
	   [RFC2406]      Kent, S. and R. Atkinson, "IP Encapsulating Security
	                  Payload (ESP)", RFC 2406, November 1998.
	   [RFC2409]      Harkins, D. and D. Carrel, "The Internet Key Exchange
	                  (IKE)", RFC 2409, November 1998.
	 
	 
	Chiba, et al.                Informational                     [Page 26]
	
	RFC 3576       Dynamic Authorization Extensions to RADIUS      July 2003
	
	   [RFC2434]      Narten, T. and H. Alvestrand, "Guidelines for Writing
	                  an IANA Considerations Section in RFCs", BCP 26, RFC
	                  2434, October 1998.
	   [RFC2486]      Aboba, B. and M. Beadles, "The Network Access
	                  Identifier", RFC 2486, January 1999.
	   [RFC2865]      Rigney, C., Willens, S., Rubens, A. and W. Simpson,
	                  "Remote Authentication Dial In User Service (RADIUS)",
	                  RFC 2865, June 2000.
	   [RFC2866]      Rigney, C., "RADIUS Accounting", RFC 2866, June 2000.
	   [RFC2869]      Rigney, C., Willats, W. and P. Calhoun, "RADIUS
	                  Extensions", RFC 2869, June 2000.
	   [RFC3162]      Aboba, B., Zorn, G. and D. Mitton, "RADIUS and IPv6",
	                  RFC 3162, August 2001.
	   [RFC3280]      Housley, R., Polk, W., Ford, W. and D. Solo, "Internet
	                  X.509 Public Key Infrastructure Certificate and
	                  Certificate Revocation List (CRL) Profile", RFC 3280,
	                  April 2002.
	   [RADIANA]      Aboba, B., "IANA Considerations for RADIUS (Remote
	                  Authentication Dial In User Service)", RFC 3575, July
	                  2003.
	7.2.  Informative References
	   [RFC2882]      Mitton, D., "Network Access Server Requirements:
	                  Extended RADIUS Practices", RFC 2882, July 2000.
	   [RFC2983]      Black, D. "Differentiated Services and Tunnels", RFC
	                  2983, October 2000.
	   [AAATransport] Aboba,  B. and J. Wood, "Authentication, Authorization
	                  and Accounting (AAA) Transport Profile", RFC 3539,
	                  June 2003.
	   [Diameter]     Calhoun, P., et al., "Diameter Base Protocol", Work in
	                  Progress.
	   [MD5Attack]    Dobbertin, H., "The Status of MD5 After a Recent
	                  Attack", CryptoBytes Vol.2 No.2, Summer 1996.
	   [NASREQ]       Calhoun, P., et al., "Diameter Network Access Server
	                  Application", Work in Progress.
	 
	Chiba, et al.                Informational                     [Page 27]