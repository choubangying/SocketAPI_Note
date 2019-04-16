## 域名和IP

IP地址是珍贵的资源，可能会发生变化，维持IP不变花费较高

域名相较于IP不那么稀缺，维持一个域名不变成本较低。

因此涉及到域名和IP的相互转换(由DNS完成)。



### 1. 已知域名求IP

```c
#include <netdb.h>
struct hostent *gethostbyname (const char *__name);
```

这个函数可以用已知域名或得它的IP地址

**返回值：**

| 返回值 | 含义            |
| ------ | --------------- |
| NULL   | 调用失败        |
| 非NULL | struct hostent* |

**注：**

其中struct hostent结构体的定义如下：

```c
struct hostent
{
  char *h_name;			/* Official name of host.  */
  char **h_aliases;		/* Alias list.  */
  int h_addrtype;		/* Host address type.  */
  int h_length;			/* Length of address.  */
  char **h_addr_list;		/* List of addresses from name server.  */
#ifdef __USE_MISC
# define	h_addr	h_addr_list[0] /* Address, for backward compatibility.*/
#endif
};
```

我们只用关心h_addr_list变量即可

使用时需如下使用：

```
struct hostent* host = gethostbyname("www.baidu.com");

for(int i = 0; host->h_addr_list[i]; i++)
{
    inet_ntoa(*((struct in_addr*)host->h_addr_list[i]);
}
//h_addr_list[i]保存的是struct in_addr结构体的地址，在IPV4的时候；
//struct hostent并不只是为IPV4设计的
```

### 2. 已知IP求域名

```
#include <netdb.h>
struct hostent* gethostbyaddr (const void *__addr, __socklen_t __len,
				      int __type);
```

**返回值：**

| 返回值 | 含义            |
| ------ | --------------- |
| NULL   | 失败            |
| 非NULL | struct hostent* |

此函数的结果返回只关心struct hostent的h_name变量