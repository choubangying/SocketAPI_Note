## 网络地址及其API

### 1.网络地址

一个设备想要接入网络需要首先给它一个网络地址，只有有了地址，别的设备才能找到它，也才可以区分别的设备

网络地址分两类：

1. IPv4         4byte表示网络地址
2. IPv6         6byte表示网络地址

这两者的区别在于IPv6的地址能容纳的主机数目更多。当下大部分还是IPv4的网络地址。

IPv4的4byte地址分为网络地址和主机地址，按占字节的不同分为A，B，C，D等类型：

参见：https://www.cnblogs.com/hijacklinux/p/6983718.html

### 2.端口号

一个网络地址用来区分不同的设备，但是设备里面还运行这不同的程序，端口号就用来区分同一设备下的不同程序。

- 端口号用16位二进制数表示，因此范围为0-65535。

- 0-1023被知名程序使用了，像22被ssh用了...

- 我们一般使用1024-65535之后的端口
- TCP程序和UDP程序不存在端口冲突（一个TCP的连接和可以和另一个UDP的连接使用同一个端口）

### 3.IPv4结构体

实际编程中存在两种可以表示IPv4的结构体struct sockaddr和struct sockaddr_in

struct sockaddr_in用来表达IPv4

struct sockaddr则是通用类型，并不是只表示IPv4

这两个结构体size一样，可以强转使用

#### 3.1 struct sockaddr_in结构体（IPv4专用）

```c
struct sockaddr_in
{
    sa_family_t sin_family;     /* Address family  */
    in_port_t sin_port;         /* Port number.  */
    struct in_addr sin_addr;    /* Internet address  */
    unsigned char sin_zero[8];  /* Pad to size of `struct sockaddr'.  */
};

其中struct in_addr的定义如下：
struct in_addr
{
    in_addr_t s_addr; /* 32bit ip addr */
};
```

**struct sockaddr_in成员分析：**

1. sin_family-地址簇

   | 地址簇   | 含义                           |
   | -------- | ------------------------------ |
   | AF_INET  | IPv4网络协议使用的地址簇       |
   | AF_INET6 | IPv6网络协议使用的地址簇       |
   | AF_LOCAL | 本地通信采用的UNIX协议的地址簇 |

2. sin_port-端口号

   保存16bit端口号，以***网络字节序***保存

3. sin_addr-ip地址

   保存32bit ip地址， 以***网络字节序***保存

4. sin_zero[8]-凑大小数组

   无特殊作用，只是为了struct sockaddr_in结构体和struct sockaddr结构体保持一致

#### 3.2 另一个struct sockaddr结构体（通用）

```c
struct sockaddr
{
    sa_family_t sin_family;	/* Common data: address family and length.  */
    char sa_data[14];		/* Address data.  */
};
```

### 4. 网络字节序与地址变换

#### 4.1 大小端与网络字节序

计算机在存储多字节数据时有两种存储方式：

| 存储方式              | 含义                   |
| --------------------- | ---------------------- |
| 大端（Big Endian）    | 内存高地址存数据低字节 |
| 小端（Little Endian） | 内存高地址存数据高字节 |

例如在内存中存储0x12345678这个数据：

大端：

![](C:\Users\Administrator\Desktop\1.png)

​	可以看到在内存的高地址（0x22）存储的是低字节（0x78）

小端：

![](C:\Users\Administrator\Desktop\2.png)

​	可以看到在内存的高地址 （0x23）存储的是高字节（0x12）

网络数据传输多字节数据的时候也存在一个字节序的问题，例如传送0x1234这个数据，存在两种传输方式，先传0x12，再传0x34或者先传0x34，再传0x12，前一种先传输高地址数据的方式称为大端序，后一种称为小端序，

**网络字节序为大端序！！！**

#### 4.2 地址转换

既然存在网络字节序，那么不同的计算机系统可能采取不同的大小端模式，所以向网络传输的数据都应该是大端序的。网络地址和端口也是网络上需要传输的数据，因此涉及到网络地址和端口的转换。

以下几个函数完成字节序转换的工作：

```c
uint32_t ntohl (uint32_t __netlong)
uint16_t ntohs (uint16_t __netshort)
uint32_t htonl (uint32_t __hostlong)
uint16_t htons (uint16_t __hostshort)
/*
    上述函数名中h表示host（主机），n表示net（网络）
    函数名最后的l表示long，s表示short，意思是把long或者short转为网络或者主机字节序
*/
```

socket通信整个过程中，需要做字节序转换的就是往struct sockaddr_in填充数据的时候，其它时刻（发送数据）都是自动完成的，无需我们关心。

### 5. 网络地址初始化与分配

#### 5.1 字符串与大端序的整形地址 互转

```c
#include <arpa/inet.h>

/* Convert Internet host address from numbers-and-dots notation in CP
   into binary data in network byte order.  */
in_addr_t inet_addr (const char *__cp)
    
/* Convert Internet host address(binary data in network byte order) to 
numbers-and-dots notation in CP.  */   
char *inet_ntoa (struct in_addr __in)
```

#### 5.2 网络地址初始化

```c
/* server端 */
#define HOSTIP "127.0.0.1"
#define HOSTPORT 5678

struct sockaddr_in addr;

memset(&addr, 0, sizoeof(addr));           /* 初始化addr */
addr.sin_family = AF_INET;                 /* 设置地址簇 */
addr.sin_addr.s_addr = inet_addr(HOSTIP);  /* 设置你通过哪个IP提供服务 */
addr.sin_port = htons(HOSTPORT);           /* 设置你通过哪个端口提供服务 */
......
/* 其它操作 */
......
    
/* client端 */
#define SERVIP "127.0.0.1"
#define SERVPORT 5678

struct sockaddr_in addr;

memset(&addr, 0, sizoeof(addr));           /* 初始化addr */   
addr.sin_family = AF_INET;                 /* 设置地址簇 */
addr.sin_addr.s_addr = inet_addr(SERVIP);  /* 设置你想要连接的服务器IP */
addr.sin_port = htons(SERVPORT);           /* 设置你你想要连接的服务器应用的端口 */
......
/* 其它操作 */
......
```

INADDR_ANY ：

这个宏用于简化server端网络地址初始化，使用这个宏就不必吗，每次输入ip地址：

```
struct sockaddr_in addr;

memset(&addr, 0, sizeof(addr));
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = htonl(INADDR_ANY);
addr.sin_port = htons(5678);
```

