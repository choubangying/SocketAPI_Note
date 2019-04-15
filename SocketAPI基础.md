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

示例：

```c
int serv_sock;
......
/* 其它操作 */
......
serv_sock = socket(PF_INET, SOCK_STREAM, 0);
if(serv_sock == -1)
{
    printf("socket() err!!!\n");
    exit(-1);
}
......
```



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

​	之前创建的socket文件描述符

**参数2 __addr：**

​	存有地址信息（ip，port）的结构体，需注意地址信息初始化（转换字节序），参见其它文档

**参数3 __len：**

​	__addr的长度（大小）

示例：

```c
int serv_sock;
struct sockaddr_in addr;

memset(&addr, 0, sizeof(addr));
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = htonl(INADDR_ANY);
addr.sin_port = htons(5678);

//get a socket
serv_sock = socket(PF_INET, SOCK_STREAM, 0);
if(serv_sock == -1)
{
    printf("socket() err!!!\n");
    exit(-1);
}

//bind
if(bind(serv_sock, (struct sockaddr*)&addr, sizeof(addr)) == -1)
{
    printf("bind() err!!!\n");
    exit(-1);
}
......
......
```



### 二。Client端API







