# UNIX 本地域套件字 AF_UNIX 实现进程间通信的网络应用越来越多
基于 socket 的框架上发展出一种 IPC 机制，就是 UNIX Domain Socket，它不需要经过网络协议栈，不需要打包拆包，在本地进程间通讯地安全性和效率上要比网络 socket 好很多。

UNIX Domain Socket 提供面向数据包和面向流的两种 API 接口，类似于 TCP 和 UDP。
## 服务端大致流程
~~~C
    server_fd = socket(AF_UNIX, SOCK_STREAM, 0);
    server_addr.sun_family = AF_UNIX;
    strcpy(server_addr.sun_path, "your-server-name");
    bind(server_fd, (struct sockaddr *)&server_addr, len);
    listen(server_fd, 5);
    client_fd = accept(server_sfd, (struct sockaddr *)& server_addr, len);
}
~~~
## 客户端大致流程
~~~C 
    client_fd = socket(AF_UNIX, SOCK_STREAN, 0);
    server_addr.sun_family = AF_UNIX;
    strcpy(server_addr.sun_path, "your-server-name");
    connect(client_fd, (struct sockaddr *)&server_addr, len);
~~~