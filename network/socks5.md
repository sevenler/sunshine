###Socks5 代理协议  
Socket5是一个代理协议，支持TCP、UDP，很多软件都支持，著名的Shadowsocks就是基于Socket5协议实现的，其中名字中的socks就是Socket5。

Socket5协议一共分为三个部分：
1.安全验证：完成Socket5的连接安全校验
2.建立连接：完成代理的请求链接，并确认状态
3.数据传输

######安全验证(Auth)
1.客户端请求Socket5代理服务，并带上自己支持的Socket5协议
请求的数据包的格式如下：
```
+----+----------+----------+
|VER | NMETHODS | METHODS  |
+----+----------+----------+
| 1  |    1     | 1 to 255 |
+----+----------+----------+

VER  协议版本，0x05
NMETHODS METHODS的数量
METHODS
    0x00 无验证需求
    0x01 通用安全服务应用程序接口(GSSAPI)
    0x02 用户名/密码(USERNAME/PASSWORD)
    0x03 至 X'7F' IANA 分配(IANA ASSIGNED)
    0x80 至 X'FE' 私人方法保留(RESERVED FOR PRIVATE METHODS)
    0xFF 无可接受方法(NO ACCEPTABLE METHODS)
```  

2.Socket5代理回复自己的验证方式
```
+----+--------+
|VER | METHOD |
+----+--------+
| 1  |   1    |
+----+--------+
VER  协议版本，0x05
METHOD  支付的验证方式
```

3.客户端根据不同的验证方式，进入子协议，并发送验证数据包  
根据上一步骤接受到的验证方式，分别做不同的操作  
    1.0x00, 无须任何操作
    2.0x01, 关于通用安全服务应用程序接口，请看另外一篇文章
    3.0x02,用户名/密码验证,则发送如下数据包：
```
+----+------+----------+------+----------+
|VER | ULEN |  UNAME   | PLEN |  PASSWD  |
+----+------+----------+------+----------+
| 1  |  1   | 1 to 255 |  1   | 1 to 255 |
+----+------+----------+------+----------+

VER  协议版本，0x05
ULEN 是UNAME的长度
UNAME  用户名
PLEN 是PASSWD的长度
PASSWD 密码
```
Socket5服务返回：
```
+----+--------+
|VER | STATUS |
+----+--------+
| 1  |   1    |
+----+--------+
VER  协议版本，0x05
STATUS
    0    SUCCESS
    非0    FAILED
```

######建立连接(Connect)
1.客户端发送请求的地址以及端口
```
+----+-----+-------+------+----------+----------+
|VER | CMD |  RSV  | ATYP | DST.ADDR | DST.PORT |
+----+-----+-------+------+----------+----------+
| 1  |  1  | X'00' |  1   | Variable |    2     |
+----+-----+-------+------+----------+----------+

#其中各个字段
VER  协议版本，0x05
CMD  
CONNECT        0x01
BIND           0x02
UDP ASSOCIATE  0x03
RSV      保留字段
ATYP     地址类型
IPV4         0x01
HOSTNAME     0x03
IPV6         0x04
DST.ADDR   请求服务器的域名或者IP地址
DST.PORT   请求服务器的端口
```

关于CMD的描述：
```
CONNECT : 0x01, 建立代理连接
BIND : 0x02,告诉代理服务器监听目标机器的连接,也就是让代理服务器创建socket监听来自目标机器的连接。FTP这类需要服务端主动联接客户端的的应用场景。
    1.只有在完成了connnect操作之后才能进行bind操作
    2. bind操作之后，代理服务器会有两次响应, 第一次响应是在创建socket监听完成之后，第二次是在目标机器连接到代理服务器上之后。
UDP ASSOCIATE : 0x03, udp 协议请求代理。
```

2.Socket5代理根据地址以及端口建立连接，并返回连接状态
```
+----+-----+-------+------+----------+----------+
|VER | REP |  RSV  | ATYP | BND.ADDR | BND.PORT |
+----+-----+-------+------+----------+----------+
| 1  |  1  | X'00' |  1   | Variable |    2     |
+----+-----+-------+------+----------+----------+

VER  协议版本，0x05
REP  连接远程主机的状态
    X00 succeeded
    X01 general SOCKS server failure
    X02 connection not allowed by ruleset
    X03 Network unreachable
    X04 Host unreachable
    X05 Connection refused
    X06 TTL expired
    X07 Command not supported
    X08 Address type not supported
    X09 to X'FF' unassigned
RSV      保留字节
ATYP     地址类型
BND.ADDR  连接远程主机的地址
BND.PORT  连接远程主机的端口
```
绑定的地址和端口根据请求中的CMD的不同而不同
```
CONNECT: 此时绑定的地址是指代理服务器连接到目标机器时的ip和端口
BIND: 这里会有两次响应
第一次响应中是指代理服务器创建监听socket时绑定的ip和端口
第二次响应中是指代理服务器监听的scoket收来自目标机器连接时的ip和端口
UDP ASSOCIATE: 此时绑定地址指明了客户发送UDP消息至服务器的端口和地址
```

######数据传输(Relay)
Socket5代理将转发客户端的请求数据包，并回传远端服务器返回的数据包给客户端  
如果是TCP连接，则直接转发。如果是UDP连接，需要做如下操作：
???   

######参考链接:  
1.[socket5 协议学习与实现(一)](http://www.mojidong.com/network/2015/03/07/socket5-1/)  
2.[rfc1928](https://www.ietf.org/rfc/rfc1928.txt)  
3.[rfc1929](https://www.ietf.org/rfc/rfc1929.txt)  
4.[rfc1961-socks5-gssapi](https://github.com/shadowsocks/redsocks/blob/master/doc/rfc1961-socks5-gssapi.txt)  
