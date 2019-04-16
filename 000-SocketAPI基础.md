## SocketAPI基础

### 一。Server端基础API:

#### 1. socket函数

```c
#include <sys/socket.h>
int socket (int __domain, int __type, int __protocol)
```

此函数用来创建socket。

**返回值：**

| 返回值 | 含义                                |
| ------ | ----------------------------------- |
| -1     | socket创建失败                      |
| 非-1   | socket文件描述符（Linux一切皆文件） |

**参数1 __domain，用来选择使用哪个协议簇 ：**

| 名称      | 协议簇               |
| --------- | -------------------- |
| PF_INET   | IPV4互联网协议簇     |
| PF_INET6  | IPV6互联网协议簇     |
| PF_LOCAL  | 本地通信的UNIX协议簇 |
| PF_PACKET | 底层套接字的协议簇   |
| PF_IPX    | IPX Novell协议簇     |

**参数2 __type，套接字类型 ：**

套接字在传输数据的时候对数据的传输方式有相应的规范，常见的有： 

1. SOCK_STREAM -> 面向连接的套接字
   - 传输过程数据不会丢失
   - 按序传输
   - 传输的数据无边界
2. SOCK_DGRAM -> 面向消息的套接字
   - 强调快速传输，传输次序不一定正确
   - 传输的数据可能损毁或者丢失
   - 传输的数据有边界
   - 传输的数据有大小限制

### 2. bind函数

```c
#include <sys/socket.h>
int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)
```

此函数用来捆绑此服务程序ip和port到socket文件描述符，也就是指定__fd使用本机哪一个ip和port

**返回值：**

| 返回值 | 含义     |
| ------ | -------- |
| 0      | bind成功 |
| -1     | bind失败 |

**参数1 __fd：**

​	之前创建的服务端socket文件描述符

**参数2 __addr：**

​	存有地址信息（ip，port）的结构体，需注意地址信息初始化（转换字节序），参见其它文档

**参数3 __len：**

​	__addr的长度（大小）

### 3. listen函数

```c
#include <sys/socket.h>
int listen (int __fd, int __n)
```

此函数用来在绑定了addr信息的socket文件描述符上监听连接，三次握手在此处完成

**返回值：**

| 返回值 | 含义 |
| :----: | :--: |
|   0    | 成功 |
|   -1   | 失败 |

**参数1 __fd:**

服务器socket文件描述符

**参数2 __n:**

排队的最大个数（两队列之和）

**注：**

内核会为每一个绑定地址监听的socket文件描述符维护两个队列

1. 未完成连接队列 --> 处于三次握手状态的放在此队列

2. 已完成连接队列 --> 完成三次握手状态的放在此队列

### 4. accept函数

```c
#include <sys/socket.h>
int accept (int __fd, struct sockaddr* __addr, socklen_t *__restrict __addr_len);
```

此函数做如下事：

1. 从**已完成连接队列**里取"客户端对象"，并从参数返回客户端的sockaddr相关信息
2. accept成功，就返回由内核创建的全新fd用来代表和客户端的连接，之后的数据收发都通过此fd

**返回值：**

| 返回值 |        含义        |
| :----: | :----------------: |
|  非-1  | 有效的文件描述符fd |
|   -1   |        失败        |

**参数1 __fd：**

服务器套接字的文件描述符fd

**参数2 __addr：**

获得的客户端连接的sockaddr信息，出参

**参数3 __addr_len：**

第二个参数的长度，这是一个value-result的参数，调用者先初始化它成保存__addr长度的指针，函数返回时里面保存实际的长度；

### 连贯示例：

```c++
void* ThreadAccpetor(void *args)
{
    int clientfd;
    struct sockaddr_in clientaddr;
    socklen_t clientaddrlen = sizeof(clientaddr);
    
    //create socket
    int listenfd = socket(PF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
        printf("create socket error!!!\n");
        return NULL;
    }

    // bind listenfd
    struct sockaddr_in bindaddr;
    bindaddr.sin_family = AF_INET;
    bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    bindaddr.sin_port = htons(5000);

    if (bind(listenfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) == -1)
    {
        printf("bind addr error!!!\n");
        return NULL;
    }

    // listen
    if (listen(listenfd, SOMAXCONN) == -1)
    {
        printf("listen error!!!\n");
        return NULL;
    }

    // set reuse addr and port(prevent timewait)
    int on = 1;
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, (char*)&on, sizeof(on));
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEPORT, (char*)&on, sizeof(on));

    // accept connect
    while (1)
    {
        printf("wocao!!!\n");
        clientfd = accept(listenfd, (struct sockaddr*)&clientaddr, &clientaddrlen);
		//...
    }
    
    close(listenfd);
}
```

### 二。Client端API

### 1. socket函数

同server端一致

### 2. connect函数

```c
#include <sys/socket.h>
int connect (int __fd, const struct sockaddr* __addr, socklen_t __len);
```

此函数用来向服务端建立连接。

**返回值：**

| 返回值 |   含义   |
| :----: | :------: |
|   0    | 连接成功 |
|   -1   | 连接失败 |

**参数1 __fd:**

客户端socket文件描述符

**参数2 __addr：**

连接的服务器的地址信息

**参数3 __len：**

第二个参数的长度

### 连贯示例：

```c
#include <sys/socket.h>
#include <arpa/inet.h>

#define SERVERIP "192.168.0.102"
#define SERVERPORT 3000

int main(int argc, const char* argv[])
{
    int clientfd;
    struct sockaddr_in serv_addr;
    
    /* 1 */
    clientfd = socket(PF_INET, SOCK_STREAM, 0);
    if( clientfd == -1 )
    {
        Log("socket() err!!!");
    }

    memset(&serv_addr, 0, sizeof(struct sockaddr_in));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(SERVERIP);
    serv_addr.sin_port = htons(SERVERPORT);

    /* 2 */
    if( connect(clientfd, (struct sockaddr*)(&serv_addr), sizeof(serv_addr)) == -1 )
    {
        close(clientfd);
        Log("connect() err!!!");
    }

    while(1)
    {
        /* read/write data from server */
    }
    
    close(clientfd);
    return 0;
}
```







