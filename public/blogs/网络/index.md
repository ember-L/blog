# Socket网络编程



## C/S模型

服务器-客户机，即Client-Server(C/S)结构。C/S结构通常采取两层结构。服务器负责数据的管理，客户机负责完成与用户的交互任务。

在C/S结构中，应用程序分为两部分:服务器部分和客户机部分。服务器部分是多个用户共享的信息与功能，执行后台服务，如控制共享数据库的操作等;客户机部分为用户所专有，负责执行前台功能，在出错提示、在线帮助等方面都有强大的功能，并且可以在子程序间自由切换。

![CS Model](https://ember-img.oss-cn-chengdu.aliyuncs.com/CS%20Model.png)

## IP层

每台互联网中的主机都运行实现`TCP/IP`协议，并且所有现代计算都支持这个协议。互联网中的客户端与服务端混合使用套接字(socket)接口函数与Unix的I/O函数进行通信。

### ip地址

一个ip地址是32位无符号整数。如下述所示网络程序将IP地址存放在一个结构体中

```c
/* IP address structure */
struct in_addr {
	uint32_t s_addr; /* Address in network byte order (big-endian) */
};
```

### 字节序

字节序分为大端字节序（big endian）和小端字节序（little endian）。

- **大端字节序(Big-Endian)**：是指一个整数的高位字节（23～31 bit）存储在内存的低地址处，低位字节（0 ～7 bit）存储在内存的高地址处。人们通常书写字节序就是大端序。
- **小端字节序(Little-Endian)**：是指整数的高位字节存储在内存的高地址处，而低位字节则存储在内存的低地址处。这是计算机中读取内存的方式。

计算机的内部处理都是**小端字节序**。在计算机内部，小端序被广泛应用于现代 CPU 内部存储数据，而在其他场景，比如**网络传输**和**文件存储**则使用**大端序**。

![Byte_order1](https://ember-img.oss-cn-chengdu.aliyuncs.com/Byte_order1.png)

> 字节序转换函数

互联网中的主机有不同的字节顺序，TCP/IP为任意整数数据项定义了**统一**的网络字节序(Network byte order)，采用**大端序**(big-endian)。在IP地址结构中存放的地址总是以大端序存放的，即使在主机使用小端序的字节序，Unix同样也提供了字节序转换的函数。

```c
#include <arpa/inet.h>
/*主机字节序转网络字节序 host to net*/
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

/*网络字节序转主机字节序 net to host*/
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(unit16_t netshort);
```



> 32位字节序转换的实现

```c
#define htonl(x)	__bswap_32 (x)
#define ntohl(x)	__bswap_32 (x)
/* Swap bytes in 32-bit value.  */
#define __bswap_constant_32(x)					\
  ((((x) & 0xff000000u) >> 24) | (((x) & 0x00ff0000u) >> 8)	\
   | (((x) & 0x0000ff00u) << 8) | (((x) & 0x000000ffu) << 24))
```

在socket库中提供的字节序的转换主要是使用了**位操作**来实现(实际也能够使用指针实现)，使用与操作获取1字节，做位移操作，以下图的大端序的高字节`0x4`为例，使用 `0xff000000u`取出这个字节并向右移动24位，也就是将高字节移动了低字节，最后将剩余的3字节做不同的位移操作，并进行与操作就能实现交换的目的了。

![Byte_order](https://ember-img.oss-cn-chengdu.aliyuncs.com/Byte_order.png)



> 判断字节序类型

```c
#include<stdio.h>
void byteorder(){
    union{
        short value;
        char union_bytes[sizeof(short)];
    }test;				//union共用数据域
    test.value=0x0102;	//大端序写法
    // 01 02
    if((test.union_bytes[0]==1) && (test.union_bytes[1]==2)){
        printf("big endian\n");
    }else if((test.union_bytes[0]==2) && (test.union_bytes[1]==1)){
        printf("little endian\n");
    }else{
        printf("unknown...\n");
}
```



> IP地址转点分十进制串                                                                                                                                                                                          

- `inet_pton`：将点分十进制(src)转换为一个二进制网络字节顺序的IP地址。如果src不合法，函数返回0，其他错误返回-1并设置errno的值。
- `inet_ntop`：将二进制网络字节序IP地址转换为对应的点分十进制表示，并将以null结尾的字符串复制到dst中。
- `inet_addr`：将传入的点分十进制(cp)转为二进制网络字节顺序的IP地址作为返回值返回。

n：代表网络(network)、p：代表表示(presentation)

```c
#include <arpa/inet.h>
int inet_pton(AF_INET, const char *src, void *dst);

const char *inet_ntop(AF_INET, const void *src, char *dst,
												socklen_t size);

in_addr_t inet_addr(const char *cp);
```



> 调用方式

```c
struct sockaddr_in server_addr;
//将点分十进制的ipv4地址，转换为整数网络字节序地址
inet_pton(AF_INET,"127.0.0.1",&server_addr.sin_addr);
/*等同于下述表达式*/
server_addr.sin_addr = inet_addr("127.0.0.1");

char buf[100];
//将整数网络字节序地址，转换为点分十进制的ipv4地址
inet_ntop(AF_INET,&server_addr.sin_addr,buf,100);
```



## 套接字接口

套接字接口(socket interface)就是**一组函数**，将其与Unix I/O结合起来便可以创建网络应用。

BSD Socket APIs（Berkeley Software Distribution Socket APIs），是面向 Userspace Application 的接口封装层，提供了一套兼容绝大部分网络通信协议族的标准 Socket APIs。

- `socket()`：创建一个新的 socket，返回一个 int 类型的 socket fd（File Descriptor，套接字文件描述符），用于后续的网络连接操作。
- `bind()`：将 socket 与一个本地 IP:Port  绑定，通常用于**服务端**，以便在本地监听网络连接。
- `listen()`：开始监听来自远程主机的连接请求，通常用于**服务器端**，等待来自客户端的连接请求。
- `accept()`：接受一个连接请求，返回一个新的 socket fd，通常用于**服务器端**，用于接收客户端的连接请求。
- `connect()`：建立与远程主机的连接，通常用于**客户端**，以便连接到远程服务器。
- `close()`：关闭 socket 连接。

> TCP网络应用模型：

![socketAPI](https://ember-img.oss-cn-chengdu.aliyuncs.com/socketAPI.png)

将上述TCP socket创建比喻成电话系统则是这样

> Server：接听方

- `socket`：让接听方有部电话手机
- `bind`：告诉别人接听方的电话号码
- `listen`：当别人拨打你电话时，接听方可以听到电话铃声响起
- `accept`：点击接听，接听到电话

> client：拨打方

- `socket`：让拨打方有部电话手机
- `connect`：拨打指定的电话，建立通话

### 通用套接字地址结构

从Unix内核角度来看，套接字就是一个通信的端点。从unix程序角度来看就是一个文件，用于相应的文件描述符。

套接字地址存放在在16字节的结构体中，这个结构体有不同的变体，1.通用型式的套接字地址、2.专用型的套接字地址。

> 通用套接字结构体

```c
/* socket地址结构体 */
struct sockaddr {
    uint16_t sa_family; /* Protocol family */
    char sa_data[14]; /* Address data */
};
```



- `sa_family`成员是地址族类型（sa_family_t）的变量。**地址族**类型通常与**协议族**类型对应，如下表所示：

| 协议族   | 地址族   | 描述           |
| -------- | -------- | -------------- |
| PF_UNIX  | AF_UNIX  | UNIX本地协议族 |
| PF_INET  | AF_INET  | TCP/IPv4协议族 |
| PF_INET6 | AF_INET6 | TCP/IPv6协议族 |

宏`PF_*`和`AF_*`都定义在`bits/socket.h`头文件中，且后者与前者有完全相同的值，所以二者通常可以混用。

- `sa_data`成员用于存放**socket地址值**。但是，不同的协议族的地址值 具有不同的含义和长度，如下表所示：

| 协议族   | 地址值含义与长度                                             |
| -------- | ------------------------------------------------------------ |
| PF_UNIX  | 文件路径名，长度可达108字节                                  |
| PF_INET  | 16bit端口号 + 32bit IPv4地址，共6字节                        |
| PF_INET6 | 16bit端口号 + 32bit流标识+128 bit IPv6地址+32bit范围ID,共126字节 |

由上表可知，`sa_data`字段无法容纳多数的协议族地址值，因此在linux下又定义了一个新的通用socket结构：

```c
#include<bits/socket.h>
struct sockaddr_storage{
    sa_family_t sa_family;
    unsigned long int__ss_align;
    char__ss_padding[128-sizeof(__ss_align)];
};
```

### 专用套接字地址结构

上面这两个**通用socket地址结构体**显然很不好用，比如设置与获取 IP地址和端口号就需要执行烦琐的位操作。所以Linux为各个协议族提 供了专门的socket地址结构体。

> 专用socket地址：IPv4套接字

- `sin_family`：协议族类型，通常情况位AF_INET，表示IPv4地址。
- `sin_port`：2字节(16位)的端口号。以大端序存储。
- `sin_addr`：4字节(32位)的IP地址。以大端序存储。

​	通过观察下述结构体，IP套接字将通用套接字的` sa_data`字段的前6字节进行划分出来，设置成了端口号(2字节)以及IP地址(4字节)，剩余的8字节用于填充结构体字节长度，以便与通用套接字的内存字节相同。

```c
/* IP socket address structure */
struct sockaddr_in {//_in后缀代表internet，而不是input
    uint16_t sin_family; 		/* 协议族，AF_INET*/
    uint16_t sin_port; 			/*端口号*/
    struct in_addr sin_addr; 	/* IP地址 */
    unsigned char sin_zero[8]; /* 这8字节内存不使用，用于扩充结构体字节*/
};

struct in_addr{
	u_int32_t s_addr;/*IPv4地址，要用网络字节序表示*/
};
```



> 通用套接字地址的作用是什么？

​	在套接字API函数(如`connect`、`bind`、`accept`)的设计中，都需要一个指向与协议相关的套接字地址。那么该如何定义套接字函数参数，使之能够接收**各种类型**的套接字地址结构？在现在我们是可以使用void指针，根据协议类型，再到函数内部进行特定的类型强转，但是void指针是C89版才引入的。

​	所以设计者的解决办法是设计一个**通用**套接字地址结构(sockaddr)作为**函数参数**，根据要求与协议再设计**特定**套接字地址结构，这样程序便可以将**与协议特定套接字结构**(`sockaddr_in`)强转为通用套接字地址，这样就可以作为参数，进行函数调用了。



> 专用socket地址：IPv6套接字

```c
struct sockaddr_in6{
    sa_family_t sin6_family;/*地址族：AF_INET6*/
    u_int16_t sin6_port;/*端口号，要用网络字节序表示*/
    u_int32_t sin6_flowinfo;/*流信息，应设置为0*/
    struct in6_addr sin6_addr;/*IPv6地址结构体，见下面*/
    u_int32_t sin6_scope_id;/*scope ID，尚处于实验阶段*/
};
struct in6_addr{
	unsigned char sa_addr[16];/*IPv6地址，要用网络字节序表示*/
};
```



> IPv6套接字地址结构的字节长度比通过地址结构大，那么类型强转会出现问题吗？

在上述的了解中，特定的套接字的内存字节长度有的是比通用套接字内存字节长度大的(IPv6套接字地址`sockaddr_in6` = 28字节，通用套接字地址`sockaddr` = 16字节)，在这种情况下我们对特定套接字结构进行类型强转，从直觉上是会出现问题的。但为什么可以这样实现呢？

首先我们要知道bind、connet、accept的通用套接字地址参数都是**指针类型**

如`int bind(int sockfd,const struct sockaddr *addr,socklen_t addrlen)`函数。

如果对指针的原理稍微了解一点，就可以知道，在64位系统中指针的字节长度为8字节，无论什么数据类型它的指针都是8字节，指针实际上就是一个64位无符号整型数的地址值。因此这里的指针类型是不是通用套接字并不重要。

因此对指针进行**强制类型**转换后只是**访问数据的方式不同**了，比如说`int`指针一次访问4字节的内存、`char`指针一次访问字节。对于结构体更是不同，我们使用`->`引用结构体成员，就可以访问结构体不同的字段。这样我们便可以通过通用套接字的协议簇字段`sa_family`，进行将指针类型强转成特定协议套接字了。



想要了解更多，可以阅读笔者写的另一篇文章[指针详解]()

### 创建套接字：socket

> 简介

在Unix/Linux中所有的资源可以视作为**文件**，包括套接字也不例外，它就是可读、可写、可控制、可关闭的**文件描述符**。

创建空白的**socket**描述符，像`creat`系统调用一样，创建一个文件描述符。

`socket`函数返回的描述符只是**部分打开**的，**不能用于读写**。如何打开套接字，还是要取决与程序是服务端还是客户端。

> 函数接口

客户端与服务端使用`socket`来创建一个套接字描述符(`sockfd`)：

```c
#include<sys/types.h>
#include<sys/socket.h>
int socket(int domain,int type,int protocol);
```

参数的解释如下所示：

- `domain`：指定**协议族类型**，对于`TCP/IP`协议族，该参数被设置为PF_INET或PF_INET6，对于Unix协议族有PF_UNIX。
- `type`：指定**服务类型**，主要有两种服务类型`SOCK_STREAM`(流服务，表示使用TCP协议)和`SOCK_UGRAM`(数据报服务，表示使用UDP服务)。
- `protocol`：在前两个参数构成的协议集合下，再选择一个具体的协议。不过这个值通常都是唯一的。可选：`IPPROTO_TCP`、`IPPTOTO_UDP`，大部分情况下我们都将其指定为0，表示默认协议。

**函数返回值**：成功：套接字描述符`sockfd`。失败：-1，并设置`errno`错误码。

> setsockopt：设置套接字选项值，解决TIME_OUT问题

```c
#include <sys/socket.h>

int setsockopt(int sockfd, int level, int optname, 
               				const void *optval, socklen_t optlen);
```

参数的解释如下所示：

- `sockfd` ：指定 socket fd。
- `level`：指定选项的协议层，可选 `SOL_SOCKET`、`IPPROTO_TCP`、`IPPROTO_IP` 等。
- `optname` ：指定要设置的选项名。
  - `SO_REUSEADDR`：int 类型，表示重用 IP 地址。
  - `SO_KEEPALIVE`：int 类型，用于启用/禁用 Keepalive（保持连接）功能。
  - `SO_LINGER`：`struct linger` 类型，用于指定关闭套接字时的行为。
  - `TCP_NODELAY`：int 类型，用于禁用 Nagle 算法，从而实现数据的实时传输。
- `optval` ：指定存放选项值的缓冲区入口。
- `optlen` ：指定选项值缓冲区的长度。

**函数返回值**：成功：0。失败：-1，并设置`errno`错误码。



### 命名套接字：bind

> 函数接口

**服务端**使用`bind`函数将服务端的socket地址绑定到`socket`函数创建的`sockfd`上：

```c
#include<sys/socket.h>
int bind(int sockfd,const struct sockaddr*my_addr,socklen_t
														addrlen);
```



参数的解释如下所示：

- `sockfd`：通过socket函数获取的**空白socket描述符**。
- `my_addr`：socket地址，这个地址将会分配给sockfd进行绑定。
- `addrlen`：socket地址结构的字节长度。

**函数返回值**：成功：0。失败：-1，并设置`errno`错误码。

> 作用

​	将socket地址绑定到空白socket描述符，通过`socket`函数创建一个socket描述符(`sockfd`)。但是这个`sockfd`只是指定了协议族，并没有指定具体的socket地址。因此我们需要将`sockfd`与socket地址绑定，这被称作为**命名套接字**。



> 服务端

​	在服务器中，需要对socket进行命名，因此需要将socket地址与socket描述符绑定，只有这样客户端才可以连接到服务器。

> 客户端

​	客户端通常是不需要命名socket，采用匿名的方式与服务器进行连接，使用系统自动分配的套接字地址。因此在**客户端**程序的设计中，我们不需要调用`bind`去命名socket。

### 监听套接字：listen

> 函数接口

**服务端**将`sockfd`转换成`listenfd`，作为客户端发起连接请求的端点：

```c
#include<sys/socket.h>
int listen(int sockfd,int backlog);
```

参数的解释如下所示：

- `sockfd`：执行过`bind`函数的socket描述符
- `backlog`：指定内核**监听队列的最大长度**，监听队列的长度如果超过backlog，服务端将不受理新的客户连接，客户端也将收到ECONNREFUSED错误信息。

**函数返回值**：成功：0。失败：-1，并设置`errno`错误码。

> 简介

**服务端**中调用`listen`函数是告诉内核，描述符是被服务端使用的。将socket描述符从一个主动套接字转化为一个监听套接字(listening socket),该`listenfd`可以接收来自客户端的请求。

客户端是发起请求的**主动实体**，服务端是等待请求的**被动实体**。默认情况，内核会认为`socket`创建的描述符是**主动套接字**(active socket)，它存在与一个连接的客户端。

### 接收连接：accept

> 函数接口

**服务端**通过调用`accept`函数来等待来自客户端的连接请求：

```c
#include<sys/socket.h>
int accept(int sockfd,struct sockaddr*addr,socklen_t*addrlen);
```

参数的解释如下所示：

- `sockfd`：执行过`listen`系统调用的监听描述符
- `addr`：通过建立连接，获取得到的客户端socket地址
- `addrlen`：指定上述客户端socket地址结构字节长度。

**函数返回值**：成功：客户端`sockfd`。失败：-1，并设置`errno`错误码。

> 作用

​	`accept`函数**等待**(会阻塞服务端程序)客户端的连接请求到达监听描述符`listenfd`，获取客户端的`socket`地址，并返回一个已连接描述符(connected descriptor)，这个描述符可用于Unix /Linux的I/O函数与客户端进行通信。到这里服务端的socket套接字就完全打开了可以用于读写了。



> 监听套接字与已连接套接字的区别

- `listenfd`：客户端**请求连接**的端点。通常被**创建一次**，并存在与服务端的整个生命周期。
- `connfd`：服务端与客户端**建立起连接**的一个端点。服务器每接收一次连接创建一次，存在于服务端服务于客户端的整个过程中。

有了这两者的区别，使得我们可以创建并发服务器，同时的处理多个客户端连接。



### 发起连接：connect

> 函数接口

**客户端**通过调用`connect`函数来建立和服务器的连接，connect的调用会导致程序阻塞，直到连接成功建立或是发生错误。

```c
#include<sys/socket.h>
int connect(int sockfd,const struct sockaddr*serv_addr,socklen_t
															addrlen);
```

参数的解释如下所示：

- `sockfd`：客户端系统调用`socket`函数创建的socket描述符
- `serv_addr`：服务器监听的socket地址
- `addrlen`：指定`serv_addr`的地址长度

`connect`成功时返回0。一旦成功建立连接，sockfd就唯一地标识了 这个连接，客户端就可以通过读写sockfd来与服务器通信。connect失败则返回-1并设置errno。



> 监听描述符与已连接描述符建立过程

1. 服务端调用`accpet`阻塞，等待`listenfd`的连接请求
2. 客户端通过调用`connect`阻塞程序，向服务端发出连接请求。
3. 服务端从`accept`返回得到`connfd`。客户端从`connect`返回，此时的`clientfd`已经与`connfd`建立起连接。两个描述符都可以进行IO操作与通信了。

![role](https://ember-img.oss-cn-chengdu.aliyuncs.com/role.png)



### 关闭连接

> 函数接口：close

关闭一个socket描述符释放其资源使用close，与关闭文件描述相同的方式相同。

```c
#include<unistd.h>
int close(int fd);
```

参数的解释如下所示：

- `fd`：是待关闭的socket

`close`系统调用并非总是立即关闭一个连接，而是将`fd`的**引用计数**减1。只有当fd的引用计数为0时，才真正关闭连接。多进程程序中，一次fork系统调用默认将使父进程中打开 的socket的引用计数加1，因此我们必须在父进程和子进程中都对该`sockfd`执行`close`调用才能将连接关闭。

> 函数接口：shutdown

如果无论如何都要立即终止连接（而不是将socket的引用计数减 1），可以使用如下的`shutdown`系统调用（相对于close来说，它是专门为网络编程设计的）

```c
#include<sys/socket.h>
int shutdown(int sockfd,int howto)
```

参数的解释如下所示：

- `sockfd`：待关闭的socket。
- `howto`：决定了shutdown的行为。由下表所示

| 可选值    | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| SHUT_RD   | 关闭sockfd上读的这一半。应用程序不能再针对socket文件描述符执行读操作，并且该socket接收缓冲区中的数据都被丢弃 |
| SHUT_WR   | 关闭sockfd上写的这一半。sockfd的发送缓冲区中的数据会在真正关闭连接之前全部发送出去，应用程序不可再对该socket文件描述符执行写操作。这种情况下，连接处于半关闭状态。 |
| SHUT_RDWR | 同时关闭sockfd上的读和写                                     |

`shutdown`能够分别关闭socket上的读或写，或者都关闭。而close在关闭连接时只能将socket上的读和写**同时关闭**。

`shutdown`成功时返回0，失败则返回-1并设置errno





> TCP 三次握手的解读

![TCP_three-way_handshake_socket](https://ember-img.oss-cn-chengdu.aliyuncs.com/TCP_three-way_handshake_socket.png)

1. 客户端的协议栈向服务器端发送了 `SYN` 包，并告诉服务器端当前发送序列号 j，客户端进 入 `SYNC_SENT` 状态； 
2. 服务器端的协议栈收到这个包之后，和客户端进行 `ACK` 应答，应答的值为 j+1，表示对 `SYN` 包 j 的确认，同时服务器也发送一个 `SYN` 包，告诉客户端当前我的发送序列号为 k， 服务器端进入 `SYNC_RCVD` 状态； 
3. 客户端协议栈收到 `ACK` 之后，使得应用程序从 `connect` 调用返回，表示客户端到服务器端 的单向连接建立成功，客户端的状态为 `ESTABLISHED`，同时客户端协议栈也会对服务器端 的 `SYN` 包进行应答，应答数据为 k+1；
4. 应答包到达服务器端后，服务器端协议栈使得 `accept` 阻塞调用返回，这个时候服务器端到 客户端的单向连接也建立成功，服务器端也进入 `ESTABLISHED` 状态。

## 数据传输

既然之前说过套接字就是文件，那么我们当然可以使用普通IO函数例如write、read读写函数用于TCP、UDP的数据传输。这里我们就不过多介绍了，接着主要介绍的是专门的socket数据传输函数。

### TCP数据传输

`recv`和 `send`函数，用于在 TCP Socket 中进行数据读写，属于阻塞式 I/O（Blocking I/O）模式，即：如果没有可读数据或者对端的接收缓冲区已满，则函数将一直等待直到有数据可读或者对端缓冲区可写。

> recv函数接口

```c
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

参数的解释如下所示：

- `sockfd` ：指定要接收 TCP 数据的 Socket 文件描述符。
- `buf`：指定接收数据缓冲区的入口地址。
- `len` ：指定要接收的数据的 Byte 数目。
- `flags`：指定接收数据时的选项，常设为 0。

**函数返回值**：成功：返回接收的字节数。失败：返回 -1。



> send函数接口

```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

参数的解释如下所示：

- `sockfd` ：指定要发送 TCP 数据的 Socket 文件描述符。
- `buf` ：指定发送数据缓冲区入的口地址。
- `len` ：指定要发送数据的 Byte 数目。
- `flags` ：指定发送数据时的选项，常设为 0。

**函数返回值**：成功：返回发送的字节数。失败：返回 -1。

###  UDP数据传输

`recvfrom`和 `sendto` 函数，用于在 UDP Socket 中进行数据读写以及获取对端地址。这两个函数在使用时需要指定对端的 `IP:Port`。

> recvfrom函数接口

```c
#include <sys/socket.h>

ssize_t recvfrom(int sock, void *buf, size_t nbytes, int flags,
                 			struct sockadr *from, socklen_t *addrlen);
```

参数的解释如下所示：

- `sock`：指定要接收 UDP 数据的 Socket 文件描述符。
- `buf`：指定接收数据缓冲区的入口地址。
- `nbytes`：指定要接收数据的 Byte 数目。
- `flags`：指定接收数据时的选项，常设为 0。
- `from`：指定源地址 sockaddr 结构体变量的地址。
- `addrlen`：指定 from 参数使用的长度，使用 sizeof() 获取。

**函数返回值**：成功：返回接收的字节数。失败：返回 -1。



> sendto函数接口

```c
#include <sys/socket.h>

ssize_t sendto(int sock, void *buf, size_t nbytes, int flags,
               					struct sockaddr *to, socklen_t addrlen);
```

- `sock`：指定要发送 UDP 数据的 Socket 文件描述符。
- `buf`：指定发送数据缓冲区的入口地址。
- `nbytes`：指定要发送数据的 Byte 数目。
- `flags`：指定发送数据时的选项，常设为 0。
- `to`：指定目标地址 sockaddr 结构体变量的地址。
- `addrlen`：指定 to 参数使用的长度，使用 sizeof() 获取。

**函数返回值**：成功：返回发送的字节数。失败：返回 -1。

### recvmsg 和 sendmsg

recvmsg() 和 sendmsg() 函数，用于在 TCP 和 UDP Socket 中进行数据读写，不仅可以读写数据，还可以读写对端地址、辅助数据等信息。

> recvmsg函数接口

```c
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```

- `sock` ：指定要接收 TCP 或 UDP 数据的 Socket 文件描述符。
- `msg` ：指示将接收的数据存储到 msghdr 结构体中。
- `flags` ：支持函数的行为，可选 0 或者 MSG_DONTWAIT 等标志位。

**函数返回值**：成功：返回接收的字节数。失败：返回 -1。



> sendmsg函数接口

```c
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```

- `sock` ：指定要发送 TCP 或 UDP 数据的 Socket 文件描述符。
- `msg` ：指示 msghdr 结构体中包含了要发送的数据、数据长度等信息。
- `flags` ：支持函数的行为，可选 0 或者 MSG_DONTWAIT 等标志位。

**函数返回值**：成功：返回发送的字节数。失败：返回 -1。



> msghdr 结构体定义如下：

```c
struct msghdr {
    /* 指定接收或发送数据的对端地址，可以为 NULL 或 0，表示不需要使用对端地址。*/
    void         *msg_name;       /* optional address */
    socklen_t     msg_namelen;    /* size of address */

    /* 指定接收或发送数据的缓冲区和缓冲区大小，可以使用多个缓冲区同时接收或发送数据。*/
    struct iovec *msg_iov;        /* scatter/gather array */
    size_t        msg_iovlen;     /* # elements in msg_iov */

 /* 指定一些附加的控制信息，可以为 NULL 或 0。*/
    void         *msg_control;    /* ancillary data, see below */
    size_t        msg_controllen; /* ancillary data buffer len */

 /* 指定函数的行为，例如是否需要接收带外数据等。*/
    int           msg_flags;      /* flags on received message */
};
```



> flags 参数类型

- `MSG_PEEK`：允许从接收队列中查看数据而不将其删除。这意味着，如果接收队列中有数据，recv() 函数将返回数据的一个副本，但是该数据仍将留在接收队列中。这对于查看接收队列中的数据而不实际处理它们非常有用。此外，使用 MSG_PEEK 选项，我们可以检查套接字缓冲区中是否有足够的数据可供读取，以便稍后调用 recv() 函数。
- `MSG_WAITALL`：如果套接字缓冲区中没有足够的数据，则 recv() 函数将一直等待，直到收到请求的数据量。
- `MSG_DONTWAIT`：指定此标志后，recv() 函数将立即返回，即使没有收到数据也不会阻塞。如果没有数据可用，则 recv() 将返回 -1，并将 errno 设置为 EAGAIN 或 EWOULDBLOCK。
- `MSG_OOB`：用于处理带外数据，即紧急数据。带外数据不遵循正常的传输控制协议（如 TCP），可以使用此标志将其标记为紧急数据并将其与其他数据分开处理。
- `MSG_TRUNC`：如果接收缓冲区中的数据比接收缓冲区长度长，则截断数据并返回。
- `MSG_CTRUNC`：如果接收缓冲区中的控制消息（例如带外数据或错误消息）比接收缓冲区长度长，则截断消息并返回。

## 网络信息API

### getnameinfo()

用于将一个Socket地址 转换为对应的主机名(Hostname，ip地址字符串)或服务名(Service name，端口号字符串)，以便于记录日志或者显示给用户。

> 函数接口

```c
#include <sys/socket.h>
int getnameinfo(const struct sockaddr *addr, socklen_t addrlen,
                char *host, socklen_t hostlen,
                char *serv, socklen_t servlen, int flags);
```

- `addr`：表示需要转换的Socket地址；
- `addrlen`：表示该 Socket addr址的长度；
- `host`：输出 Hostname 的存储空间。
- `serv`：输出 Service name 的存储空间。
- `hostlen`：Hostname 存储空间的大小。
- `servlen`：Service name 存储空间的大小。
- `flags`：标志参数，通常设置为 0。

**函数返回值**：成功：返回 0。失败：返回非 0，并更新 errno 全局变量。



## 实例

> talk is cheap，show you my code

接着我们用实际的案例的来展示实现简单的网络应用(echo server)，主要实现的是TCP服务器以及UDP服务器。

### TCP网络程序

> TCP API模型

![TCP_echo](https://ember-img.oss-cn-chengdu.aliyuncs.com/TCP_echo.png)



> TCP服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <netdb.h>        // for getnameinfo
#include <fcntl.h>        
#include <netinet/tcp.h> //for marco TCP_NODELAY

#include <arpa/inet.h>   //for inet_addr
#include <sys/socket.h>

#define SERVERPORT 6666
#define MAXLINE 100
#define BUF_LEN 100

/*简单的echo实现*/
void echo(int clientfd){
    char buf[BUF_LEN];
    ssize_t recv_len = 0;
    while(1){
        /*接收指定Client Socket发出的数据*/
        memset(buf,0,sizeof(buf));
        if((recv_len = recv(clientfd,buf,BUF_LEN,0))<= 0){
            printf("disconnect client fd = %d\n",clientfd);
            /*处理完Client请求，关闭连接*/
            close(clientfd);
            break;
        }
        printf("Recevice data from client: %s", buf);
        /*发送接收到的信息给客户端*/
        send(clientfd,buf,recv_len,0);
    }
}

int createTCPServer(int port){
    /*配置Server Sock信息*/
    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    
    //INADDR_ANY = 0.0.0.0监听本机所有地址
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(port);

    /*创建Server Socket*/
    int serverfd = 0;
    if((serverfd = socket(AF_INET,SOCK_STREAM,0))== -1){
        printf("Create socket file descriptor ERROR\n");
        exit(-1);
    }

    int yes = 1;
    //SO_REUSEADDR防止TIME_WAIT问题，参考smallchat
    setsockopt(serverfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(yes));
    
    if(bind(serverfd,
            (struct sockaddr*)&server_addr,
            sizeof(server_addr))==-1)
    {
        printf("Bind socket ERROR.\n");
        exit(-1);
    }

    if(listen(serverfd,10) == -1){
        printf("Listen socket ERROR.\n");
        exit(-1);     
    }
    return serverfd;
}

int acceptClient(int listenfd){
    /*初始化Client socket 信息*/
    struct sockaddr_storage client_addr;
    bzero(&client_addr, sizeof(client_addr));
    socklen_t clientlen = sizeof(client_addr);
    int connfd = 0;

	char client_hostname[MAXLINE] , client_port[MAXLINE];
    if((connfd = accept(listenfd,
                    (struct sockaddr*)(&client_addr),
                    (socklen_t *)&clientlen)) == -1)
    {
        printf("Accept connection from client ERROR.\n");
        return -1;   
    }
    getnameinfo((struct sockaddr*)&client_addr,clientlen,client_hostname,MAXLINE,
                                                            client_port,MAXLINE,0);
    printf("Connected to (%s %s)\n",client_hostname,client_port);

    return connfd;
}


int main(int argc, char ** argv){
    int listenfd = createTCPServer(SERVERPORT);
    int clientfd = 0;
    
    while(1){
        if((clientfd = acceptClient(listenfd)) < 0){
            exit(-1);
        }
        echo(clientfd);
    }
    /*关闭监听描述符*/
    close(listenfd);
    return 0;
}
```



> 简单的TCP客户程序

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_LEN 100

int main(void)
{
    /* 配置 Server Sock 信息。*/
    struct sockaddr_in srv_sock_addr;
    memset(&srv_sock_addr, 0, sizeof(srv_sock_addr));
    srv_sock_addr.sin_family = AF_INET;
    srv_sock_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    srv_sock_addr.sin_port = htons(6666);

    int clientfd = 0;
    char send_buff[BUF_LEN];
    char recv_buff[BUF_LEN];

    /* 创建 Client Socket。*/
    if ((clientfd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == -1){
        printf("Create socket ERROR.\n");
        exit(-1);
    }

    /* 连接到 Server Sock 信息指定的 Server。*/
    if (connect(clientfd,
                        (struct sockaddr *)&srv_sock_addr,
                        sizeof(srv_sock_addr)) == -1){
        printf("Connect to server ERROR.\n");
        exit(EXIT_FAILURE);
    }

    /* 死循环从终端接收输入，并发送到 Server。*/
    while (1) {
        /* 从 stdin 接收输入，再发送到建立连接的 Server Socket。*/
        fputs("Send to server> ", stdout);
        memset(send_buff, 0, BUF_LEN);
        fgets(send_buff, BUF_LEN, stdin);
        send(clientfd, send_buff, BUF_LEN, 0);

        /* 从建立连接的 Server 接收数据。*/
        memset(recv_buff, 0, BUF_LEN);
        int n = 0;
        if((n = recv(clientfd, recv_buff, BUF_LEN,0)) <= 0){
            printf("read error\n");
            break;
        }
        printf("Recevice from server: %s\n", recv_buff);
    }
    /* 每次 Client 请求和响应完成后，关闭连接。*/
    close(clientfd);

    return 0;
}
```



> 运行程序：

Makefile文件:

```makefile
CC = gcc
CFLAGS = -Wall -O -fno-omit-frame-pointer -ggdb -gdwarf-2
OBJ = tcp_client tcp_server

all:tcp_client.c tcp_server.c
	$(CC) $(CFLAGS) tcp_client.c -o tcp_client 
	$(CC) $(CFLAGS) tcp_server.c -o tcp_server

.PHONY : clean
clean:
	rm $(OBJ)
```

新建两个终端：一个运行`make`命令编译文件后，并运行`./tcp_server`程序，另一个终端运行`./tcp_client`程序，在客户端程序命令行中输入数据，便能够得到服务端的回复了。再打开第三个终端就使用`netstat`命令便可以查看连接状态。

观察下图，可发现客户端已**41718**的端口号与服务端建立了连接。

![netstat_tcp](https://ember-img.oss-cn-chengdu.aliyuncs.com/netstat_tcp.png)

### UDP网络程序



> UDP API模型

![UDP_echo](https://ember-img.oss-cn-chengdu.aliyuncs.com/UDP_echo.png)



> 服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <netdb.h>        // for getnameinfo
#include <fcntl.h>        
#include <netinet/tcp.h> //for marco TCP_NODELAY

#include<arpa/inet.h>   //for inet_addr
#include<sys/socket.h>

#define SERVERPORT 6666

const int MAXLINE = 100;
const int BUF_LEN = 100;

/*简单的echo实现*/
void echo(int clientfd){
    char buf[BUF_LEN];
    ssize_t recv_len = 0;
    while(1){
        /*接收指定Client Socket发出的数据*/
        memset(buf,0,sizeof(buf));
        if((recv_len = recv(clientfd,buf,BUF_LEN,0))<= 0){
            printf("disconnect client fd = %d\n",clientfd);
            /*处理完Client请求，关闭连接*/
            close(clientfd);
            break;
        }
        printf("Recevice data from client: %s", buf);
        send(clientfd,buf,recv_len,0);
    }
}

int createTCPServer(int port){
    /*配置Server Sock信息*/
    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    
    //INADDR_ANY = 0.0.0.0监听本机所有地址
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(port);

    /*创建Server Socket*/
    int srvfd = 0;
    if((srvfd = socket(AF_INET,SOCK_STREAM,0))== -1){
        printf("Create socket file descriptor ERROR\n");
        exit(-1);
    }

    int yes = 1;
    //SO_REUSEADDR防止TIME_WAIT问题
    setsockopt(srvfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(yes));// Best effort.

    if(bind(srvfd,
            (struct sockaddr*)&server_addr,
            sizeof(server_addr))==-1)
    {
        printf("Bind socket ERROR.\n");
        exit(-1);
    }

    if(listen(srvfd,10) == -1){
        printf("Listen socket ERROR.\n");
        exit(-1);     
    }
    return srvfd;
}

int acceptClient(int listenfd){
    /*初始化Client socket 信息*/
    struct sockaddr_storage client_addr;
    bzero(&client_addr, sizeof(client_addr));
    socklen_t clientlen = sizeof(client_addr);
    int connfd = 0;

	char client_hostname[MAXLINE] , client_port[MAXLINE];
    if((connfd = accept(listenfd,
                    (struct sockaddr*)(&client_addr),
                    (socklen_t *)&clientlen)) == -1)
    {
        printf("Accept connection from client ERROR.\n");
        return -1;   
    }
    getnameinfo((struct sockaddr*)&client_addr,clientlen,client_hostname,MAXLINE,
                                                            client_port,MAXLINE,0);
    printf("Connected to (%s %s)\n",client_hostname,client_port);

    return connfd;
}


int main(int argc, char ** argv){

    int listenfd = createTCPServer(SERVERPORT);

    int clientfd = 0;
    while(1){
        if((clientfd = acceptClient(listenfd)) < 0){
            exit(-1);
        }
        echo(clientfd);
    }
    /*关闭监听描述符*/
    close(listenfd);
    return 0;
}
```





> 客户端

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_LEN  100

int main(void)
{
    int clientfd;
    char buf[BUF_LEN] = {0};
    struct sockaddr server_addr;
    socklen_t addr_size = 0;
    struct sockaddr_in  server_sock_addr;
    
    /* 创建客户端socket */
    if (-1 == (clientfd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP)))
    {
        printf("socket error!\n");
        exit(1);
    }
    
    /* 向服务器发起请求 */
    memset(&server_sock_addr, 0, sizeof(server_sock_addr));  
    server_sock_addr.sin_family = AF_INET;
    server_sock_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_sock_addr.sin_port = htons(6666); // 6666为服务端开启的端口
    
    addr_size = sizeof(server_addr);
    
    while (1)
    {
        fprintf(stdout,"Send to server> ");
        fgets(buf,BUF_LEN,stdin);
        /* 发送数据到服务端 */
        sendto(clientfd, buf, strlen(buf), 0, 
                (struct sockaddr*)&server_sock_addr, sizeof(server_sock_addr));
        
        /* 接受服务端的返回数据 */
        recvfrom(clientfd, buf, BUF_LEN, 0, &server_addr, &addr_size);
        printf("receive from server: %s\n", buf);
        
        memset(buf, 0, BUF_LEN);   // 重置缓冲区
    }
    
    close(clientfd);   // 关闭套接字
 
    return 0;
}
```





## 总结

以上就是比较简单的C语言套接字编程了，虽然Unix提供socket API特别很多，但是在编写完简单的UDP与TCP网络程序后，相信你能够对socket网络接口有大致的了解。熟能生巧，keep going！！！



> 参考：

1. [linux：Socket 网络框架与编程示例 (qq.com)](https://mp.weixin.qq.com/s/IZVv8CqhajDix746x2AYTA)
2. [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/3e/labs.html)

