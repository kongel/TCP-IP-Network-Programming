# TCP-IP-Network-Programming
Tcp/ip network programming by 尹圣雨(Korea)

# 简介
本书很简单,适合新人上手
通俗易懂,很多简单小例子
新手一定要正常动手写

阅读的第一本技术书
很多知识点都偏向实战开发而不是理论,与408很不同

本书提到的windows下的网络编程完全不用看,只看linux

很多技术细节完全不需要,太在意,只一笔带过或者当作前置默认知识就行了

学网络编程之前需要有linux下系统编程的基础知识

切忌焦躁,过于求成
完成一章节最少需要认真学习俩小时,放弃写这个例子,放弃那个就什么都学不到了
不要完美主义,纠结细节,更不能随便放弃知识点,过于惜力
把握好学习的原则和适度性

眼光要长远,不要纠结短期利益
一个章节也就几个小时学习时间,可能要3,4个小时了
一天纯学习时间8小时也就能学俩知识点,不要太焦躁


# 第一章 理解网络编程和套接字
socket函数创建套接字
```
#include <sys/socket.h>
int socket(int domain,int type,int protocol);
domain 套接字使用的协议族protocol family信息
type 套接字数据的传输类型信息
protocol 计算机通信协议信息

成功返回socket的文件描述符,失败返回-1
```
协议族 prorocol family
> 头文件 `sys/socket.h` 中声明的协议族
>

| 名称      | 协议族               |
| --------- | -------------------- |
| PF_INET   | IPV4 互联网协议族    |
| PF_INET6  | IPV6 互联网协议族    |
| PF_LOCAL  | 本地通信 Unix 协议族 |
| PF_PACKET | 底层套接字的协议族   |
| PF_IPX    | IPX Novel 协议族     |
PF_INET与AF_INET是等效的

套接字类型 type参数
决定了协议族并不能同时决定数据的传输方式

**面向连接的套接字 SOCK_STREAM**
- 传输数据不会丢失
- 按序传输数据
- 传输数据不存在数据边界 boundary

发送端发3次一共100字节,只要接收端缓冲数组够大,一次read就能把数据读到自己的缓冲区

**面向消息的套接字 SOCK_DGRAM**
- 强调快速传输而不是传输顺序
- 传输数据可能丢失或者销毁
- 传输数据有边界
- 限制每次传输大小
没有连接
发送端发两次,接收端就要接受两次

第三个参数 type
指定domain协议族,type套接字传输方式,第三个参数,protocol指定具体的协议信息
int tcp_socket=socket(PF_INET,SOCK_STREAM,IPPROTO_TCP);
满足PF_INET IPv4和tcp方式的就只有IPPROTO_TCP,传0或IPPROTO_TCP都可以

bind函数为套接字分配ip地址和端口号
```
#include <sys/socket.h>
int bind(int sockfd,struct sockaddr *myaddr,socklen_t addrlen);
成功返回0,失败返回-1
```
bind函数在实际的函数调用中
```
struct sockaddr_in serv_addr;
bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr))
```
用到的各种结构体实际的定义
```
struct sockaddr_in
{
    sa_family_t sin_family;  //地址族（Address Family）
    uint16_t sin_port;       //16 位 TCP/UDP 端口号,用网络字节序保存
    struct in_addr sin_addr; //32位 IP 地址,用网络字节序保存,就是一个uint32_t
    char sin_zero[8];        //不使用
};

struct in_addr
{
    in_addr_t s_addr; //32位IPV4地址
}
```
关于以上两个结构体的一些数据类型。

| 数据类型名称 |             数据类型说明             | 声明的头文件 |
| :----------: | :----------------------------------: | :----------: |
|   int 8_t    |           signed 8-bit int           | sys/types.h  |
|   uint8_t    |  unsigned 8-bit int (unsigned char)  | sys/types.h  |
|   int16_t    |          signed 16-bit int           | sys/types.h  |
|   uint16_t   | unsigned 16-bit int (unsigned short) | sys/types.h  |
|   int32_t    |          signed 32-bit int           | sys/types.h  |
|   uint32_t   | unsigned 32-bit int (unsigned long)  | sys/types.h  |
| sa_family_t  |       地址族（address family）       | sys/socket.h |
|  socklen_t   |       长度（length of struct）       | sys/socket.h |
|  in_addr_t   |       IP地址，声明为 uint_32_t       | netinet/in.h |
|  in_port_t   |       端口号，声明为 uint_16_t       | netinet/in.h |

bind函数用到的第二个参数,先声明为sockaddr_in,填充完数据后再强制类型转化为sockaddr类型
sockaddr_in内sa_family_t sin_family;可以取得值为

> 地址族

| 地址族（Address Family） | 含义                               |
| ------------------------ | ---------------------------------- |
| AF_INET                  | IPV4用的地址族                     |
| AF_INET6                 | IPV6用的地址族                     |
| AF_LOCAL                 | 本地通信中采用的 Unix 协议的地址族 |



listen函数设置监听状态
```
#include <sys/socket.h>
int listen(int sockfd,int backlog);
成功返回0,失败返回-1
```

accept函数接受连接
```
#include <sys/socket.h>
int accept(int sockfd.struct sockaddr * addr,socklen_t *addrlen);
成功返回文件描述符,失败返回-1
```

请求连接函数,这里调用的是客户端套接字,connect函数
```
#include <sys/socket.h>
int connect(int sockfd,struct sockaddr *serv_addr,socklen_t addrlen);
成功返回0,失败返回-1
```
## 1.2 基于linux的基础文件操作
linux下socket也被视为文件的一种,可以直接使用文件io相关的函数

|文件描述符|	对象|
|----|----|
|0	标准输入：|Standard Input|
|1	标准输出：|Standard Output|
|2	标准错误：|Standard Error|

这里注意区分库函数和系统调用
STDIN_FILENO是文件描述符,在unistd.h定义  用read函数处理
stdin是FILE* 是stdio.h内的标准Io     用c语言库函数fread处理

打开文件函数open
```
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *path,int flag)
path是打开文件的文件名和路径信息,由字符串表示
flag表示文件打开模式
```
第二个参数flag如果需要多个参数传递,应该通过位或运算
文件打开模式如下表：

| 打开模式 |            含义            |
| :------: | :------------------------: |
| O_CREAT  |       必要时创建文件       |
| O_TRUNC  |      删除全部现有数据      |
| O_APPEND | 维持现有数据，保存到其后面 |
| O_RDONLY |          只读打开          |
| O_WRONLY |          只写打开          |
|  O_RDWR  |          读写打开          |

关闭文件
```
#include <unistd.h>
int close(int fd);
参数为要关闭的文件或者套接字的文件描述符
```

write写入文件
```
#include <unistd.h>
ssize_t write(int fd,const void* buf,size_t nbytes);
fd是文件描述符
buf是要传输数据的缓冲地址值
nbytes是要传输的字节数
```

read读取文件数据
```
#include <unistd.h>
ssize_t read(int fd,void* buf size_t nbytes);
buf:接收数据的缓冲地址值
nbytes:要接收数据的最大字节数
```

## 本章提到的服务端hello world程序
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int serv_sock;
    int clnt_sock;

    struct sockaddr_in serv_addr;
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size;

    char message[] = "Hello World!";

    if (argc != 2)
    {
        printf("Usage : %s <port>\n", argv[0]);
        exit(1);
    }
    //调用 socket 函数创建套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if (serv_sock == -1)
        error_handling("socket() error");

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(atoi(argv[1]));
    //调用 bind 函数分配ip地址和端口号
    if (bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)
        error_handling("bind() error");
    //调用 listen 函数将套接字转为可接受连接状态
    if (listen(serv_sock, 5) == -1)
        error_handling("listen() error");

    clnt_addr_size = sizeof(clnt_addr);
    //调用 accept 函数受理连接请求。如果在没有连接请求的情况下调用该函数，则不会返回，直到有连接请求为止
    clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_addr, &clnt_addr_size);
    if (clnt_sock == -1)
        error_handling("accept() error");
    //稍后要将介绍的 write 函数用于传输数据，若程序经过 accept 这一行执行到本行，则说明已经有了连接请求
    write(clnt_sock, message, sizeof(message));
    close(clnt_sock);
    close(serv_sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```


## hello world client程序
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int sock;
    struct sockaddr_in serv_addr;
    char message[30];
    int str_len;

    if (argc != 3)
    {
        printf("Usage : %s <IP> <port>\n", argv[0]);
        exit(1);
    }
    //创建套接字，此时套接字并不马上分为服务端和客户端。如果紧接着调用 bind,listen 函数，将成为服务器套接字
    //如果调用 connect 函数，将成为客户端套接字
    sock = socket(PF_INET, SOCK_STREAM, 0);
    if (sock == -1)
        error_handling("socket() error");

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_addr.sin_port = htons(atoi(argv[2]));
    //调用 connect 函数向服务器发送连接请求
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)
        error_handling("connect() error!");

    str_len = read(sock, message, sizeof(message) - 1);
    if (str_len == -1)
        error_handling("read() error!");

    printf("Message from server : %s \n", message);
    close(sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

atoi函数
把字符串数字转化为int数字
```
#include <stdlib.h>
int atoi(const char *str);

```

## 课后习题
Linux 中，对套接字数据进行 I/O 时可以直接使用文件 I/O 相关函数；而在 Windows 中则不可以。原因为何？

答：Linux把套接字也看作是文件，所以可以用文件I/O相关函数；而Windows要区分套接字和文件，所以设置了特殊的函数

# 第二章 套接字类型与协议设置
讲了数据边界和几个函数参数
都在第一章讲了

## 习题

&lt; 就是<
&gt; 就是 >

tcp传输和udp传输的区别

# 第三章 地址族与数据序列

端口号不能重复
但是tcp套接字和udp套接字不会共用端口号,允许重复

网络字节序是大端序
intel和amd的cpu都采用小端序
数据字节序转换是默认的
## 字节序转换函数
```
unsigned short htons(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```
通过函数名称掌握其功能，只需要了解：

htons 的 h 代表主机（host）字节序。
htons 的 n 代表网络（network）字节序。
s 代表 short
l 代表 long

把字符串ip信息转化为网络字节序整数
```
#include <arpa/inet.h>
in_addr_t inet_addr(const char *string);
成功返回大端32位整数,失败返回INADDR_NONE
```

和上面的完全一样,就是把字符串ip化为32位整数
```
#include <arpa/inet.h>
int inet_aton(const char *string, struct in_addr *addr);
/*
成功时返回 1 ，失败时返回 0
string: 含有需要转换的IP地址信息的字符串地址值
addr: 将保存转换结果的 in_addr 结构体变量的地址值
*/
```

把网络字节序的ip转化为字符串ip
```
#include <arpa/inet.h>
char *inet_ntoa(struct in_addr adr);
```
需要注意的是这个函数返回一个指针


## 习题
ipv4是4个字节
ipv6是16字节

ip地址区分主机
port端口号区分程序

















