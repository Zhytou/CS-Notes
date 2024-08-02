# Socket

- [Socket](#socket)
  - [Socket API](#socket-api)
    - [TCP Socket](#tcp-socket)
    - [UDP Socket](#udp-socket)
  - [I/O Model](#io-model)
    - [Blocking I/O Server](#blocking-io-server)
    - [I/O Multiplexing](#io-multiplexing)

套接字（Socket）实际是位于传输层和应用层之间的一个编程接口。

## Socket API

为了执行网络I/O，一个进程必须做的第一件事就是调用socket函数，并指定期望的通信协议类型去创建一个socket。当socket函数成功时，它返回一个非负整数，与文件描述符类似，我们将其称为套接字描述符(Socket Descriptor)，一般简写为sockfd。

```c++
#include<sys/socket.h>
int socket(int family, int type, int protocol);
```

其中，family参数指定协议簇，type指定套接字类型，protocol指定协议。它的取值分别如下。

- AP_INET：
- AP_INET6：
- AP_LOCAL
- AP_ROUTE

- SOCK_STREAM
- SOCK_DGRAM
- SOCK_SEQPACKET
- SOCK_RAW

- IPPROTO_TCP
- IPPROTO_UDP
- IPPROTO_STCP

### TCP Socket

![TCP Socket建立通信的过程](../img/computer_network_tcp_socket.png)

### UDP Socket

![UCP Socket建立通信的过程](../img/computer_network_udp_socket.png)

## I/O Model

### Blocking I/O Server

```c++
int main() {
    int serverSocket, clientSocket;
    struct sockaddr_in serverAddr, clientAddr;
    socklen_t clientAddrLen = sizeof(clientAddr);
    
    // 创建socket
    serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    
    // 设置服务器地址和端口
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(80);
    
    // 绑定端口
    bind(serverSocket, (struct sockaddr*) &serverAddr, sizeof(serverAddr));
    // 监听端口
    listen(serverSocket, 5);
    
    while (1) {
        // 接受客户端连接
        clientSocket = accept(serverSocket, (struct sockaddr*) &clientAddr, &clientAddrLen);
        
        // 创建一个新线程处理客户端请求
        int id = fork();
        if (id == 0) {

        }
    }
    
    close(serverSocket);
    
    return 0;
}
```

### I/O Multiplexing

```c++
```
