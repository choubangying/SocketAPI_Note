## TCP半关和可选项相关API

### TCP半关API-shutdown函数

TCP建立起连接后有两条流通道。

- 输入流通道
- 输出流通道

当我们需要单向关闭某个流通道时可以使用shutdown函数

```c
#i8nclude <sys/socket.h>
int shutdown (int __fd, int __how)
```

**返回值：**

| 返回值 | 含义 |
| :----: | :--: |
|   0    | 成功 |
|   -1   | 失败 |

**参数1 __fd：**

你要操作的socket文件描述符

**参数2 __how：**

- SHUT_RD         -> 断开输入流，套接字无法收取数据
- SHUT_WR        -> 断开输出流，套接字无法发出数据
- SHUT_RDWR   -> 同时断开输出输入流，套接字既不能收取也不能发送数据

### 可选项相关API

socket的可选项



socket有很多选型可以被设置，使用setsockopt函数来设置

```
#include <sys/socket.h>
int setsockopt (int __fd, int __level, int __optname,
		       const void *__optval, socklen_t __optlen)
```

它用来设置socket的各种参数，包括端口复用

**返回值：**

| 返回值 | 含义 |
| :----: | :--: |
|        |      |
|        |      |





