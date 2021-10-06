# TCP协议数据流的读取
### 客户端粘包
客户端采用了Nagle优化算法，假如客户端发送abc,def,hij三个数据包，通过调用三次write来发送数据，Nagle算法可能把这三次write调用打包成一个数据包发送给服务器。
### 服务器粘包
客户端发送的三个数据包被服务器接收后，保存在服务器针对该socket连接的接收缓冲区中，等服务器调用recv接受时，可能会一次把abcdefhij全部收到，这就叫做服务器粘包。
## 解决方案
给数据包定义一个统一的格式，采用`包头+包体`的格式。包头中会记录本次数据包（包头+包体）的长度，其中包头的长度是固定的。在连接中添加数据包读取的状态标志curStat来记录当前数据包读到了哪一步了。
### 包头+包体
![Recv_Packet_包头](https://user-images.githubusercontent.com/91582638/136179453-d88f2cb5-a13f-4cef-aff2-2c1c45af4a24.png)
### 读取数据包
![Recv_Packet_收数据](https://user-images.githubusercontent.com/91582638/136180770-59242eeb-6aae-4a4c-acb3-1172b681e73b.png)
<br/>接受一个数据包的过程：<br/>
1、先收固定长度的包头；<br/>
2、根据包头的内容,计算出包体的长度<br/>
3、再根据包体长度来接受包体数据就收到了一个完整的数据包<br/><br/>
### 接受数据流程
![Receive_packet_流程图](https://user-images.githubusercontent.com/91582638/136181241-718f6711-8fdb-4540-a3ea-338a855b51c5.png)
<br/><br/>
接受一个数据包的流程：<br/>
1、epoll_wait获取ClientFd的读事件。<br/>
2、recv读取指定长度的数据。<br/>
3、根据curStat判断当前数据包读取到哪一步（收取包头还是收取包体）。<br/>
4、判断包头/包体是否收取完整，修改curStat的状态。<br/>
5、等待下一次读事件触发，继续读取数据。<br/>
