---
layout: post
title: TCP 服务端、客户端简单代码
date: 2014-10-16 9:08:12
categories: network
tags: network  tcp
excerpt:  TCP 服务端、客户端简单代码示例
---

TCP 服务端示例：

```c++
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    // 创建 socket
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == -1) {
        std::cerr << "Failed to create socket." << std::endl;
        return -1;
    }

    // 绑定 socket
    sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(8080);

    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        std::cerr << "Failed to bind socket." << std::endl;
        return -1;
    }

    // 监听端口
    if (listen(server_fd, 3) == -1) {
        std::cerr << "Failed to listen on port 8080." << std::endl;
        return -1;
    }

    std::cout << "Server started. Listening on port 8080..." << std::endl;

    // 等待客户端连接
    int client_socket;
    sockaddr_in client_addr;
    socklen_t client_addr_size = sizeof(client_addr);

    client_socket = accept(server_fd, (struct sockaddr*)&client_addr, &client_addr_size);
    if (client_socket == -1) {
        std::cerr << "Failed to accept connection." << std::endl;
        return -1;
    }

    std::cout << "Client connected. IP address: " << inet_ntoa(client_addr.sin_addr) << std::endl;

    // 接收客户端发送的数据
    char buffer[1024] = {0};
    int num_bytes = read(client_socket, buffer, sizeof(buffer));

    std::cout << "Received " << num_bytes << " bytes of data from client: " << buffer << std::endl;

    // 关闭连接
    close(client_socket);
    close(server_fd);

    return 0;
}

```

TCP 客户端示例：

```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <cstring>

int main() {
    // 创建套接字
    int client_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_socket == -1) {
        std::cerr << "Failed to create socket\n";
        return -1;
    }

    // 设置服务器地址和端口
    struct sockaddr_in server_addr{};
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    // 连接服务器
    if (connect(client_socket, (struct sockaddr *) &server_addr, sizeof(server_addr)) == -1) {
        std::cerr << "Failed to connect server\n";
        return -1;
    }

    // 发送数据
    const char *msg = "Hello, server!";
    if (send(client_socket, msg, strlen(msg), 0) == -1) {
        std::cerr << "Failed to send data\n";
        return -1;
    }

    // 关闭连接
    close(client_socket);

    return 0;
}


```

TCP 三次握手是在 `listen` 之后，`accept` 之前完成的。

在 TCP 服务器端启动后，首先需要调用 `socket()` 函数创建一个 `socket`，然后通过 `bind()` 函数绑定 `socket` 到一个地址（通常是服务器的 IP 地址和端口号）。之后，需要调用 `listen()` 函数，开始监听来自客户端的连接请求。这时，TCP 服务器处于 `LISTEN` 状态，等待客户端连接。

当客户端连接时，客户端会发送一个 `SYN` 报文，表示请求连接。此时，服务器端收到连接请求后，会发送一个 `SYN-ACK` 报文给客户端，表示确认连接请求，并向客户端发送确认号。当客户端收到 `SYN-ACK` 报文后，会发送一个 `ACK` 报文给服务器端，表示确认连接成功。

当这三次握手完成后，服务器调用 `accept()` 函数接受客户端的连接请求，此时服务器可以和客户端进行通信。

客户端的TCP三次握手是在 connect 函数之后进行的。当客户端调用 connect 函数时，它向服务器发送一个SYN报文，请求建立连接。服务器收到SYN报文后，会发送一个SYN+ACK报文给客户端，表示同意建立连接。客户端收到服务器的SYN+ACK报文后，会再次发送一个ACK报文，表示确认连接已建立。这样，TCP三次握手完成，连接就建立起来了。

从 `read` 返回 0 表示远程已关闭连接，即TCP连接已经关闭。

因为在 TCP 连接中，当一方收到对端的 FIN 报文段并返回了 ACK 报文段后，会进入 CLOSE_WAIT 状态，表示自己已经没有要发送的数据了，但是对端可能还会发送一些数据过来。当对端也没有数据要发送时，会发送一个 FIN 报文段，表示对端也已经关闭连接，这时候本地会回复一个 ACK 报文段，表示已经收到了对端的 FIN 报文段，并进入 LAST_ACK 状态。在此状态下，如果对端也确认了 ACK 报文段，那么连接就真正关闭了。此时，本地如果还继续调用 read，会返回0，表示连接已经关闭。