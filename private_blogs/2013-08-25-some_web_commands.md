#几个网络相关的命令行工具
##netcat
最大的用途就是用来处理TCP|UDP套接字
###作为端口扫描工具
nc -z url 20-100  
扫描url上的端口  
###监听传入的连接
通过这个功能再配合tar来快速有效的在服务器之间拷贝文件  
做了一个简单的测试,在服务器端运行  

    nc -l 9090 | cat -  
在客户端运行  
    
    cat tmp | nc portal 9090

不过为毛只监听到一个文件传递就自动关闭了？aha,这样的话可以用-k，强迫nc保持连接状态，即使客户端断开服务器端也不会退出。而起-k必须和-l搭配使用  
有了这个东东的话，能直接在本地给服务器传递一些命令了，而不是每次都ssh上去做事  
不过还是应该用类似rsync这样的工具来进行拷贝文件  
###可以使用netstat把任何应用通过网络暴露出来
服务器端  

    mkfifo backpipo
    nc -l 8080 0<backpipo | bash > backpipe
    
客户端  

    nc example.com 8080
这个其实没看太懂，突然想到，这个东西的安全性怎么保证，如果暴露出一个端口来，会不会所有的机器都能连接上来,需要验证一下  

###超时控制
客户端使用-w来做连接超时控制，服务器端无效  
###使用udp协议
使用参数-u来完成, udp协议和tcp协议完全忘记了。。。  

##sshuttle
首先恶补一下网络方面的知识  
###socket
从oracle的networking上看到的有关socket的定义:  
    
    A socket is one endpoint of a two-way communication link between two programs running on the network. A socket is bound to a port number so that the TCP layer can identify the application that data is destined to be sent.
    An endpoint is a combination of an IP address and a port number. Every TCP connection can be uniquely identified by its two endpoints. That way you can have multiple connections between your host and the server.

socket是两个应用之间进行通信的端点，必须和port绑定，这样tcp才能确定这个应用想传输的数据应该送给谁。这样就能看出端倪了，socket相当于应用和tcp之间的传输抽象层。  

###TCP和UDP
TCP是安全的，面向连接的，容错的传输层协议  
UPD是不满足以上三点的另外一个传输层协议  
囧rz

###ssh -TNn -D指令的研究
ssh -D的manual给出的解释是下面的。ssh在这里创建了一个socks代理服务, 在本地创建一个socket来监听相应的端口，一旦这个端口上有连接，连接就通过安全通道转向出去，同时应用层判断从哪里连接远程主机。另外，动态指的是服务端口补丁，且可能有多个。  

    Specifies a local ``dynamic'' application-level port forwarding.
    This works by allocating a socket to listen to port on the local
    side, optionally bound to the specified bind_address.  Whenever a
    connection is made to this port, the connection is forwarded over
    the secure channel, and the application protocol is then used to
    determine where to connect to from the remote machine.  *Currently
    the SOCKS4 and SOCKS5 protocols are supported, and ssh will act
    as a SOCKS server*.  Only root can forward privileged ports.
    Dynamic port forwardings can also be specified in the configura-
    tion file.

    IPv6 addresses can be specified by enclosing the address in
    square brackets.  Only the superuser can forward privileged
    ports.  By default, the local port is bound in accordance with
    the GatewayPorts setting.  However, an explicit bind_address may
    be used to bind the connection to a specific address.  The
    bind_address of ``localhost'' indicates that the listening port
    be bound for local use only, while an empty address or `*' indi-
    cates that the port should be available from all interfaces.

—T指令官方的说法是disable pseudo-tty allocation，也就是不占用shell  
-N的话，不执行远程命令，纯做端口映射用的  
-n将标准输入重定向到/dev/null中，在后台运行的时候必须要用这个东东, 不然就会很奇葩。一个trick就是用这个指令来运行x11 program  

