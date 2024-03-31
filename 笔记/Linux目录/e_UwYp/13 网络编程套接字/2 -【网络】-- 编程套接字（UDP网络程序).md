



![b43f993114487fcd20da908e02407d2](D:\未来教育考试系统V4.0\WeChat Files\wxid_ghbp1ox7z7ft22\FileStorage\Temp\b43f993114487fcd20da908e02407d2.png)



## 1. **UDP四个版本的网络程序**

### 2. **简易UDP服务端网络程序**

**一个大致的模板**



##### log.hpp

- 日记

```c++
#pragma once

#include<cstdarg>
#include<cstdio>
#include<ctime>

// 日志是有日志级别的
#define DEBUG   0
#define NORMAL  1 // 正常
#define WARNING 2 // 警告 -- 没出错
#define ERROR   3 // 错误 -- 不影响后续执行（一个功能因为条件等，没有执行）
#define FATAL   4 // 致命 -- 代码无法继续向后执行


const char* gLevelMap[] = 
{
    "DEBUG",
    "NORMAL",
    "WARNING",
    "ERROR",
    "FATAL",
};

#define LOGFILE "./threafool.log"


// 完整的日志功能，至少：日志等级 时间 日志内容 支持用户自定义
void logMessage(int level, const char* format,...) // level：日志等级; format, ...：用户传参、日志对应的信息等
{
#ifndef DEBUG_SHOW 
    if(level == DEBUG) return;
#endif 
    char stdBuffer[1024];   //标准部分
    time_t timestamp = time(nullptr); // 此处，只是想后续打印时间
    snprintf(stdBuffer,sizeof(stdBuffer),"[%s] [%ld]",gLevelMap[level],timestamp);

    char logBuffer[1024];   //自定义部分
    va_list args;
    va_start(args,format);
    // 这个时候就有一个可变参数列表的起始地址
    
     // 向缓冲区logBuffer中打印
     vsnprintf(logBuffer,sizeof(logBuffer),format,args);
     va_end(args);
     
     //向屏幕
     printf("%s %s\n",stdBuffer,logBuffer);

    // 向文件打印
    FILE* fp = fopen(LOGFILE,"a+");
    fprintf(fp,"%s%s\n",stdBuffer,logBuffer);
    fclose(fp);
}

```

---

##### **udp_server.hpp**

```cpp
#ifndef _UDP_SERVER_HPP
#define _UDP_SERVER_HPP
#include "log.hpp"
#include <string>
#include <iostream>
 
class UdpServer
{
public:
    UdpServer(uint16_t port, std::string ip = "0.0.0.0"):_port(port), _ip(ip)
    : _port(port), _ip(ip)
    {}
    
    // 从这里开始，就是新的调用，初始化服务器
    bool initServer()
    {}
 
    // 服务器开始运行
    void Start()
    {}
 
    ~UdpServer()
    {}
 
private:
    // 一个服务器，一般必须需要ip地址和port(16位的整数)
    uint16_t _port; // 端口号
    std::string _ip; // ip
};
#endif 
```

---

##### **udp_server.cc**

```c++

#include "udp_server.hpp"
#include<iostream>
#include<memory>

static void usage(std::string proc)
{
    std::cout << "\nUsage: " << proc << " ip" << " port\n" << std::endl;
}


// ./udp_server ip port 
int main(int argc,char* argv[])
{
    if(argc != 3)
    {
        usage(argv[0]);
        exit(1);
    }

    std::string ip = argv[1];
    uint16_t port = atoi(argv[2]);
    std::unique_ptr<UdpServer> svr(new UdpServer(port,ip));
    svr->initServer();
    svr->Start();
    return 0;
}
```

---

##### Makefile

```makefile
.PHONY:all 
all: udp_client udp_server 

udp_client:udp_client.cc
	g++ -o $@ $^ -std=c++11 -lpthread 
udp_server:udp_server.cc 
	g++ -o $@ $^ -std=c++11 -lpthread  

.PHONY:clean 
clean: 
	rm -r udp_client udp_server 

```

----



#### 2.1  **初始化**



##### 2.1.1 **socket**

![image-20240217213648852](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217213648852.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// 创建套接字
int socket(int domain, int type, int protocol);
```

 socket属于计算机网络，给我们提供的一个系统调用接口，其是对传输层做了相关的一层文件系统级别的封装的一个接口。



- 返回值

  >  套接字创建成功返回一个**文件描述符**，创建失败返回-1，同时错误码会被设置。



- **#问：那后续的网络的读写是否可以采用，以前的文件接口来进行操作？**

>   理论上是这样的， 但是以前的文件操作都是字节流式的，所以在UDP协议中是不适用的，只有在TCP协议那里，套接字创建好，就与文件操作一摸一样。UDP协议具有独属于自己的读写接口。
>
> -  所以，socket函数接口的返回值，当成一个**套接字 / 文件描述符**就可以了。



- **参数说明：**

>   **domain：**通常表示的是套接字的**域**（我们将来创建的套接字，是哪一种类型的套接字），也就是创建套接字的类型。
>
> ![image-20240217214459225](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217214459225.png)
>
> 我们最常用的就是：`**AF_INET、AF_UNIX, AF_LOCAL**`。其中参数说白了就是宏，就相当于struct sockaddr结构的前16个位。如果是本地通信就设置为 ***\*AF_UNIX, AF_LOCAL\**** ，如果是网络通信就设置为 ***\*AF_INET\**** （IPv4）。

>  **type：**类型，创建的套接字的通讯种类是什么。
>
>  - **融汇贯通的理解：**
>
>  >  Linux中的，文件、管道的通信类型，都叫做流式类型。而UDP的特点是面向数据报，也就是说我们可以一次一次独立的向对方在，不建立链接的情况下直接可以发送数据。
>
>   我们所填的一般就是：```SOCK_DGRAM （套接字、用户数据报套接字）。```
>
>  ```"SOCK STREAM" ```是一种套接字类型，用于在网络编程中描述一种可靠的、基于字节流的传输方式。它是在传输控制协议```（TCP）```上实现的一种套接字类型。
>
>  ```"SOCK_DGRAM" ```是另一种套接字类型，用于在网络编程中描述一种不可靠的、无连接的传输方式。它是在用户数据报协议```（UDP）```上实现的一种套接字类型。

> ​    **protocol：**
>
> 只要其前面的两个参数是什么、怎么填已经确定了，它的协议也就基本上规定好了。它是用于创建套接字的协议类别，我们可以指明为TCP或UDP，但是其会根据传入的前两个参数自动推导出我们所需要使用的是哪种协议。所以，该字段一般直接设置为0就可以了，设置为0表示的就是默认。 
>
> ![image-20240217215633112](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217215633112.png)







- 问：第一个参数与第二个参数有什么区别？

> **第一个参数：**说明了我们当前的套接字是用来进行**网络通讯**还是**本地通讯**的。
>
> **第二个参数：**如果确定是网络通讯了，那么想在网络当中**以什么方式进行通讯，**是以**数据流**还是**数据报**的方式。



---



##### 2.1.2 **bind**

![image-20240217221106041](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217221106041.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```



- **Note：**

>   使用的时候，需要将头文件带齐。
>
> ```c++
> #include <netinet/in.h>
> #include <arpa/inet.h>
> ```
>
> ​    这两个头文件，会包含我们对应的所需要用的数据类型（如：struct sockaddr），工具方法。



- **返回值：**

>   成功时，返回零。出现错误时，返回-1，并正确设置errno。



- **参数说明：**

>  **sockfd：**套接字对应的文件描述符。

>   **addr：**struct sockaddr_in类型的指针。
>
> ![img](https://img-blog.csdnimg.cn/21c15ef060904c1ca5964161d23f2ab1.png)
>
> 
>
> ```c++
> /* Structure describing an Internet socket address.  */
> struct sockaddr_in
>   {
>     __SOCKADDR_COMMON (sin_);
>     in_port_t sin_port;			/* Port number.  */  // 端口号
>     struct in_addr sin_addr;		/* Internet address.  */  // 网络地址（IP地址）
>  
>  
>     // 填充字段，不用管
>     /* Pad to size of `struct sockaddr'.  */
>     // 是一个大数组，根据不同的平台编译填充不同的大小
>     unsigned char sin_zero[sizeof (struct sockaddr) -
> 			   __SOCKADDR_COMMON_SIZE -
> 			   sizeof (in_port_t) -
> 			   sizeof (struct in_addr)];
>   };
> 
> /* Internet address.  */
> typedef uint32_t in_addr_t;
> struct in_addr
>   {
>     in_addr_t s_addr;
>   };
> ```
>
> ```c++
> // 通过宏，用##将符号拼接起来
> #define	__SOCKADDR_COMMON(sa_prefix) \
>   sa_family_t sa_prefix##family
> ```



> 对于网络地址（IP地址），比如："193.186.1.3"，称之为点分十进制风格的IP地址。由点作为分割符的每一个区域，在数字上取值范围是[0, 255]：1字节 -> 4个区域。理论上，表示一个IP地址，其实4字节就够了。4字节，每1个字节对应一个区域就行了。用字符串风格的显示，在网络通讯里没有必要，字符串风格是用于给用户看的。

-    于是需要：**点分十进制字符串风格的IP地址 <-> 4字节**。

  

     我们需要定义一个struct sockaddr_in类型的对象，提供给bind。

```c++
struct sockaddr_in local; // 是一个结构体，所以一般在使用的时候需要进行清零
```

**需要使用到的接口：** 



---s

##### 2.1.3 bzero

![image-20240217222503708](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217222503708.png)

- 其作用跟 memset 一样

```c++
#include <strings.h>
 
// 直接在一传指定字节数的内存空间当中，将数据全部进行清零
void bzero(void *s, size_t n);
```



---

##### 2.1.4  **htonl、htons、ntohl、ntohs**

![image-20240217222628850](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217222628850.png)

```c++
#include <arpa/inet.h>
 
//函数的作用是将unsigned integer从主机字节顺序转换为网络字节顺序。
uint32_t htonl(uint32_t hostlong);
 
//函数的作用是将unsigned short integer从主机字节顺序转换为网络字节顺序。
uint16_t htons(uint16_t hostshort);
 
//函数的作用是将unsigned integer从网络字节顺序转换为主机字节顺序。
uint32_t ntohl(uint32_t netlong);
 
//函数的作用是将unsigned short integer从网络字节顺序转换为主机字节顺序。
uint16_t ntohs(uint16_t netshort);
```

---

##### 2.1.5 **inet_addr**

![image-20240217222759785](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217222759785.png)



**其中的：**

`````c++
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
 
//将主机地址ip从IPv4数字和点符号转换为网络字节顺序的二进制数据。
in_addr_t inet_addr(const char *cp);
`````

---



### 3. **初始化服务器部分完成代码：**

##### udp_server.hpp

```c++
//防止头文件的多重包含
// #ifndef 表示 "if not defined"，即如果尚未定义指定的宏，则执行后续操作。
// #define _UDP_SERVER_HPP 表示如果该宏未定义，则定义它，因此后续代码将会被包含在条件指令 #ifndef 和 #endif 之间。
#ifndef _UDP_SERVER_HPP
#define _UDP_SERVER_HPP

#include"log.hpp"
#include<string>
#include<iostream>
#include<cerrno>
#include<cstring>
#include<cstdio>


//网络四件套
#include<sys/socket.h>
#include<sys/types.h>
#include<netinet/in.h>
#include<arpa/inet.h>

class UdpServer
{
public:
    // 因为是文件描述符，所以没有初始化为-1
    UdpServer(uint16_t port,std::string ip = "0.0.0") 
    :_port(port)
    ,_ip(ip)
    ,_sock(-1)
    {}

    
    // 从这里开始，就是新的调用，初始化服务器
    bool initServer()
    {
        // 1. 创造套接字  
            //网络通信就设置为  AF_INET  （IPv4）。
            //SOCK_DGRAM （套接字、用户数据报套接字）。
        _sock = socket(AF_INET,SOCK_DGRAM,0);
        if(_sock < 0)
        {
            logMessage(FATAL,"%d : %s",errno,strerror(errno));
            exit(2);
        }

         // 2. bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
         struct sockaddr_in local; // 是一个结构体，所以一般在使用的时候需要进行清零
         bzero(&local,sizeof(local)); //将数据全部清零
         local.sin_family = AF_INET;  // 即：用来表示是，本地通讯还是网络通讯

          // 因为：服务器的IP和端口未来也是要发送给对方主机的（就如同打电话，是需要知道对方的电话号码）
          // -> 也就代表需要先将数据发送到网络 -> 代表需要注意大小端问题
        local.sin_port = htons(_port); // 端口号（2字节）- 主机序列转为网络序列

        // 1. 先要将点分十进制字符串风格的IP地址 -> 4字节
        // 2. 4字节主机序列 -> 网络序列
        // 有一套接口，可以一次帮我们做完这两件事情, 让服务器在工作过程中，可以从任意IP中获取数据:inet_addr()
        local.sin_addr.s_addr = inet_addr(_ip.c_str());

        if(bind(_sock,(struct sockaddr*)&local,sizeof(local)) < 0)
        {
            logMessage(FATAL,"%d : %s",errno,strerror(errno));
            exit(2);
        }

        logMessage(NORMAL," init udp server done ... %s",strerror(errno));

        return true;
    }

    //服务器开始运行
    void Start()
    {
        
    }

    ~UdpServer()
    {
        
    }

private:
   // 一个服务器，一般必须需要ip地址和port(16位的整数)
    uint16_t _port; //端口号
    std:: string _ip; //ip地址
    int _sock; // 文件描述符 - 套接字创建成功返回一个文件描述符
};


#endif
```

---



##### udp_server.cc

```c++

#include "udp_server.hpp"
#include<iostream>
#include<memory>

static void usage(std::string proc)
{
    std::cout << "\nUsage: " << proc << " ip" << " port\n" << std::endl;
}


// ./udp_server ip port 
int main(int argc,char* argv[])
{
    if(argc != 3)
    {
        usage(argv[0]);
        exit(1);
    }

    std::string ip = argv[1];
    uint16_t port = atoi(argv[2]);
    std::unique_ptr<UdpServer> svr(new UdpServer(port,ip));
    svr->initServer();
    svr->Start();
    return 0;
}
```

---



![image-20240217232433975](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217232433975.png)



---



### 4. **开始通信**

![b43f993114487fcd20da908e02407d2](D:\未来教育考试系统V4.0\WeChat Files\wxid_ghbp1ox7z7ft22\FileStorage\Temp\b43f993114487fcd20da908e02407d2.png)



#### 4.1.1 **recvfrom**



![image-20240217233027548](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217233027548.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// 调用用于从套接字接收消息，并且无论套接字是否面向连接，都可以用于在套接字上接收数据
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```



- **返回值：**

>    这些调用返回接收的字节数，如果发生错误，则返回-1。如果发生错误，将设置errno以指示错误。



- **参数说明：**

>   **sockfd：**套接字对应的文件描述符。
>
>  	只要我们前面初始化中绑定成功了，我们就直接可以在**sockfd**中读取数据。

>   **buf，len：**代表一段缓冲区。
>
>   	我们需要进行读数据，我们就需要定义一段读取数据的缓冲区，用于存放从操作系统读取到的数据，**buf**缓冲区，**len**缓冲区读取的最大的大小。

>   **flags：**读取的方式。
>
> ​    默认为0时，代表以阻塞方式进行读取。

> **src_addr，addrlen：**（输出型参数）
>
> ​	struct sockaddr_in结构体，里面包含ip、端口号……。
>
> >  因为在当读到了别人发过来的数据时，除了数据本身最想知道的是谁发送过来的消息。以此方便我们在将数据推回别人。





---

#### 4.1.2 inet_ntoa

![image-20240217233503599](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217233503599.png)

```c++
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
 
//将网络字节顺序的二进制数据转换为主机地址IPv4数字和点符号。
char *inet_ntoa(struct in_addr in);
```

---

#### 4.1.3 **sendto**

![image-20240217233610097](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240217233610097.png)

- **返回值：**

>  本地检测到的错返回返回值-1。



- **参数说明：**

>    **sockfd：**套接字对应的文件描述符。
>
>  只要我们前面初始化中绑定成功了，我们就直接可以在**sockfd**中读取数据。

> **buf，len：**代表一段缓冲区，**buf**缓冲区，**len**缓冲区发送数据的大小。

>  **flags：**发送的方式，默认设为0。

>   **src_addr，addrlen：**代表将数据发给谁。

---

#### 4.1.4 **服务器部分完成代码：**

```cpp
    // 服务器开始运行
    void Start()
    {
        // 作为一款网络服务器，永远不退出的！
        // 服务器启动-> 进程 -> 常驻进程 -> 永远在内存中存在，除非挂了！
        // echo server: client给我们发送消息，我们原封不动返回（如同echo指令）
        char buffer[SIZE];
        for (;;)
        {
            // 注意：
            // peer：纯输出型参数
            struct sockaddr_in peer;
            bzero(&peer, sizeof(peer));
 
            // peer：输入输出型参数
            // 输入: peer 缓冲区大小
            // 输出: 实际读到的peer大小
            socklen_t len = sizeof(peer);
 
            // start.读取数据
            ssize_t s = recvfrom(_sock, buffer, sizeof(buffer) - 1, 0, (struct sockaddr *)&peer, &len);
            if (s > 0)
            {
                buffer[s] = '\0'; // 将数据当作字符串
                // 1. 输出发送的数据信息
                // 2. 是谁？？
 
                // 其实：
                // 如果我们想将数据传输回对方，直接使用peer就可以了（因为其本来就是一个填充好的字段）
                // 但是这次为了方便学习，于是将数据拿出
                uint16_t cli_port = ntohs(peer.sin_port);      // 从网络中来的！
                std::string cli_ip = inet_ntoa(peer.sin_addr); // 4字节的网络序列的IP -> 本主机的字符串风格的IP，方便显示
                printf("[%s:%d]# %s\n", cli_ip.c_str(), cli_port, buffer);
            }
 
            // 分析和处理数据（忽略）
            // end.写回数据
            sendto(_sock, buffer, strlen(buffer), 0, (struct sockaddr *)&peer, len);
        }
    }
```



---

## 5. 服务器端代码

### 5.1 udp_server.hpp

```c++
//防止头文件的多重包含
// #ifndef 表示 "if not defined"，即如果尚未定义指定的宏，则执行后续操作。
// #define _UDP_SERVER_HPP 表示如果该宏未定义，则定义它，因此后续代码将会被包含在条件指令 #ifndef 和 #endif 之间。
#ifndef _UDP_SERVER_HPP
#define _UDP_SERVER_HPP

#include"log.hpp"
#include<string>
#include<iostream>
#include<cerrno>
#include<unistd.h>
#include<cstring>
#include<cstdio>


//网络四件套
#include<sys/socket.h>
#include<sys/types.h>
#include<netinet/in.h>
#include<arpa/inet.h>

class UdpServer
{
public:
    // 因为是文件描述符，所以没有初始化为-1
    UdpServer(uint16_t port,std::string ip = "0.0.0") 
    :_port(port)
    ,_ip(ip)
    ,_sock(-1)
    {}

    
    // 从这里开始，就是新的调用，初始化服务器
    bool initServer()
    {
        // 1. 创造套接字  
            //网络通信就设置为  AF_INET  （IPv4）。
            //SOCK_DGRAM （套接字、用户数据报套接字）。
        _sock = socket(AF_INET,SOCK_DGRAM,0);
        if(_sock < 0)
        {
            logMessage(FATAL,"%d : %s",errno,strerror(errno));
            exit(2);
        }

         // 2. bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
         struct sockaddr_in local; // 是一个结构体，所以一般在使用的时候需要进行清零
         bzero(&local,sizeof(local)); //将数据全部清零
         local.sin_family = AF_INET;  // 即：用来表示是，本地通讯还是网络通讯

          // 因为：服务器的IP和端口未来也是要发送给对方主机的（就如同打电话，是需要知道对方的电话号码）
          // -> 也就代表需要先将数据发送到网络 -> 代表需要注意大小端问题
        local.sin_port = htons(_port); // 端口号（2字节）- 主机序列转为网络序列

        // 1. 先要将点分十进制字符串风格的IP地址 -> 4字节
        // 2. 4字节主机序列 -> 网络序列
        // 有一套接口，可以一次帮我们做完这两件事情, 让服务器在工作过程中，可以从任意IP中获取数据:inet_addr()
        local.sin_addr.s_addr = inet_addr(_ip.c_str());

        if(bind(_sock,(struct sockaddr*)&local,sizeof(local)) < 0)
        {
            logMessage(FATAL,"%d : %s",errno,strerror(errno));
            exit(2);
        }

        logMessage(NORMAL," init udp server done ... %s",strerror(errno));

        return true;
    }

    //服务器开始运行
    void Start()
    {
        // 作为一款网络服务器，永远不退出的！
        // 服务器启动-> 进程 -> 常驻进程 -> 永远在内存中存在，除非挂了！
        // echo server: client给我们发送消息，我们原封不动返回（如同echo指令）
        char buff[1024];

        while(true)
        {
            // 注意：
            // peer：纯输出型参数
            struct sockaddr_in peer;
            bzero(&peer,sizeof(peer));
            
            // peer：输入输出型参数
            // 输入: peer 缓冲区大小
            // 输出: 实际读到的peer大小
            socklen_t len = sizeof(peer);
            
              // start.读取数据
              ssize_t s = recvfrom(_sock,&buff,sizeof(buff)-1,0,(struct sockaddr *)&peer,&len);

              if(s > 0)
              {
                buff[s] = '\0'; //将数据当作字符串
                 // 1. 输出发送的数据信息
                 // 2. 是谁？？

                   // 其实：
                // 如果我们想将数据传输回对方，直接使用peer就可以了（因为其本来就是一个填充好的字段）
                // 但是这次为了方便学习，于是将数据拿出
                uint16_t cli_port = ntohs(peer.sin_port);  // 从网络中来的！
                std::string cli_ip = inet_ntoa(peer.sin_addr);// 4字节的网络序列的IP -> 本主机的字符串风格的IP，方便显示
                printf("[%s:%d]# %s\n", cli_ip.c_str(), cli_port, buff);
              }

              // 分析和处理数据（忽略）
              // end.写回数据
              sendto(_sock,buff,strlen(buff),0,(struct sockaddr *)&peer,len);
        }

    }

    ~UdpServer()
    {
          if (_sock >= 0)
            close(_sock);
    }

private:
   // 一个服务器，一般必须需要ip地址和port(16位的整数)
    uint16_t _port; //端口号
    std:: string _ip; //ip地址
    int _sock; // 文件描述符 - 套接字创建成功返回一个文件描述符
};


#endif
```

---

### 5.2 udp_server.cc

```c++

#include "udp_server.hpp"
#include<iostream>
#include<memory>

static void usage(std::string proc)
{
    std::cout << "\nUsage: " << proc << " ip" << " port\n" << std::endl;
}


// ./udp_server ip port 
int main(int argc,char* argv[])
{
    if(argc != 3)
    {
        usage(argv[0]);
        exit(1);
    }

    std::string ip = argv[1];
    uint16_t port = atoi(argv[2]);
    std::unique_ptr<UdpServer> svr(new UdpServer(port,ip));
    svr->initServer();
    svr->Start();
    return 0;
}
```

---



-  **netstat命令 **用于查看当前网络链接，查看本地主机当中的服务器的启动情况和未来链接信息。

  > - -n：直接打印连接的IP地址与端口信息
  > - -u：显示UDP传输协议的连线状况
  > - -p：显示正在使用SOCKET的程序识别码和程序名称
  > - -a：查看所有连接

![image-20240218013035770](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218013035770.png)

![img](https://img-blog.csdnimg.cn/55150fc2bac14f799c463e46132693aa.png)

此时，服务端就搞定了。

---



## 6. 客户端代码

### 6.1  udp_server.cc



```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
// 网络四件套
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
 
static void usage(std::string proc)
{
    std::cout << "\nUsage: " << proc << " serverIp serverPort\n"
              << std::endl;
}
 
int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        usage(argv[0]);
        exit(1);
    }
    int sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0)
    {
        std::cerr << "socket error" << std::endl;
        exit(2);
    }
 
    // 此处需要使用ip和端口所以是需要bind的。
    // 但是一般client不会显示的bind，即：程序员不会自己bind。
    // 因为如果程序员bind，就一定是手动bind：ip和端口。
    // 这样是可以跑，但是不敢这么办（不建议）。
    // 因为：client是一个客户端 -> 普通人下载安装启动使用的-> 如果程序员自己bind了：
    // client 一定bind了一个固定的ip和port，万一，其他的客户端提前占用了这个port呢？？（因为一个端口号只能一个使用）
    // 所以：client一般不需要显示的bind指定port，而是让OS自动随机选择 —— OS自动选择没有被占用的端口号
    std::string message;
    struct sockaddr_in server;
    memset(&server, 0, sizeof(server));
    server.sin_family = AF_INET;
    server.sin_port = htons(atoi(argv[2]));
    server.sin_addr.s_addr = inet_addr(argv[1]);
 
    char buffer[1024];
    while (true)
    {
        std::cout << "请输入你的信息#";
        std::getline(std::cin, message);
        if(message == "quit") break;
        // 当client首次发送消息给服务器的时候，OS会自动给client bind他的ip和port
        sendto(sock, message.c_str(), message.size(), 0, (struct sockaddr *)&server, sizeof(server));
 
        struct sockaddr_in temp;
        socklen_t len = sizeof(temp);
 
        // 读取数据
        ssize_t s = recvfrom(sock, buffer, sizeof(buffer), 0, (struct sockaddr *)&temp, &len);
        if (s > 0)
        {
            buffer[s] = 0;
            std::cout << "server echo# " << buffer << std::endl;
        }
    }
    close(sock);
 
    return 0;
}
```

---

## 7. **UDP网络程序运行**（第一版）

![img](https://img-blog.csdnimg.cn/f3dbeecc92bb45b3ab90b62b759a8f29.png)



- **简易的理解本地环回：**

> 如果客户端和服务器在一台机器上，那么将来客户端在向服务器发送消息的时候，客户端发送的消息直接经过协议栈（不会发送到网络），直接在协议栈底部直接交付给上层。
>
> ![img](https://img-blog.csdnimg.cn/089b36d677104e92b9104b8ead9b3d11.png)
>
> 
>
>  通常用于：本地网络服务器的测试。
>
> - **补充：**
>
> > 只要在**127.0.0.1**走通了，接了网络走不通，就99%是网络的问题。 





- 代码运行

![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-18_12-46-10.gif)

![image-20240218124657980](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218124657980.png)



- **Note：**

>    云服务器无法bind公网ip / 我们所指定的**非127.0.0.1** 非0.0.0.0这样的ip，也就是说一个
>
>    具体的ip在云服务器上，无法绑定。同时对于服务器来讲，也不建议bind确定的ip。
>
> ​    推荐使用任意ip的方案。
>
> ![image-20240218124833985](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218124833985.png)
>
> ![image-20240218124954245](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218124954245.png)
>
> 
>
> ```c++
> /* Address to accept any incoming messages.  */
> #define	INADDR_ANY		((in_addr_t) 0x00000000)
> ```
>
> 以此：让服务器在工作的过程中，可以从任意的ip中获取数据。有的时候一个服务器 / 计算机，有可能配的不仅仅是一张网卡，每张网卡都可能配有不同的ip，所以如果我们明确的在服务器端绑定某一个具体ip，所以该服务器就只能够收到来自于具体ip的消息。
>
>  如果采取·```INADDR_ANY，```那就是告诉操作系统，凡是发给这个主机上的指定端口的所有数据，都给'我'。而不是像之前绑定具体ip一样，根据具体ip给报文。
>
>  所以：服务器基本上100%，服务端填充ip的时候都是填充的***\*INADDR_ANY\****。
>
>   如果就是要使用具体的ip也是可以的，可以利用好三目运算符，不为空就使用我们设置入的ip。
>
>   这也就是为什么，查看中有0**.**0**.**0**.**0，其就是代表任意ip地址绑定。
>
> ![image-20240218125543115](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218125543115.png)



---



## 8. 一份好玩的代码 （第二版）

### 8.1 **执行命令**

​    前面的代码，服务端是拿到字符串，一个普通的文本，并未有多余的操作。现在如果我们将其拿到的字符串是一个命令呢？做一件事情，将发送过来的命令执行完，再将执行的结果进行返回。

---



#### 8.1.1 popen

![img](https://img-blog.csdnimg.cn/084143b6219e4dac98ecefeeb2d3a6fa.png)

```c++
#include <stdio.h>
 
FILE *popen(const char *command, const char *type);
//其会帮我们完成：
// 1、执行command -底层-> 自动先pipe()，然后fork()让子进程执行（调用exec*系列函数）command所代表的命令
// 2、返回值FILE *：可以将执行结果通过FILE*类型的指针进行读取
```

**其会帮我们完成：**

1. **command：**执行command -底层-> 自动先pipe()，然后fork()让子进程执行（调用exec*系列函数）command所代表的命令
2. **返回值FILE \*：**可以将执行结果通过FILE*类型的指针进行读取。
3. **type：**我们所打开执行的文本结果的方式。



---

#### 8.1.2 **strcasestr**

![img](https://img-blog.csdnimg.cn/03745a2fca574e71acca218e8ea4713a.png)

```c++
#include <string.h>
 
// 用于在c串haystack中查找c串needle，忽略大小写
char *strcasestr(const char *haystack, const char *needle);
// 找到了就返回：如果找到则返回needle串在haystack串中第一次出现的位置的char指针。
// 没找到就返回：NULL。
```



- udp_server.cc

  ```c++
  #include <iostream>
  #include <cstring>
  #include <unistd.h>
  
  // 网络四件套
  #include <sys/socket.h>
  #include <sys/types.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  
  using namespace std;
  static void usage(std::string proc)
  {
      std::cout << "\nUsage: " << proc << " serverIp serverPort\n"
                << std::endl;
  }
  
  // ./udp_client ip port
  int main(int argc, char *argv[])
  {
      if (argc != 3)
      {
          usage(argv[0]);
          exit(1);
      }
  
      int sock = socket(AF_INET, SOCK_DGRAM, 0);
      if (sock < 0)
      {
          std::cerr << "socket error" << std::endl;
          exit(2);
      }
  
      // 此处需要使用ip和端口所以是需要bind的。
      // 但是一般client不会显示的bind，即：程序员不会自己bind。
      // 因为如果程序员bind，就一定是手动bind：ip和端口。
      // 这样是可以跑，但是不敢这么办（不建议）。
      // 因为：client是一个客户端 -> 普通人下载安装启动使用的-> 如果程序员自己bind了：
      // client 一定bind了一个固定的ip和port，万一，其他的客户端提前占用了这个port呢？？（因为一个端口号只能一个使用）
      // 所以：client一般不需要显示的bind指定port，而是让OS自动随机选择 —— OS自动选择没有被占用的端口号
  
      std::string message;
  
      // 发送的服务端
      struct sockaddr_in server;
      memset(&server, 0, sizeof(server));
  
      server.sin_family = AF_INET;
      server.sin_port = htons(atoi(argv[2]));
      server.sin_addr.s_addr = inet_addr(argv[1]);
      cout << argv[0] << endl;
      cout << argv[1] << endl;
      cout << argv[2] << endl;
      char buffer[1024];
      while (1)
      {
          std::cout << "请输入你的信息#";
          std::getline(std::cin, message);
          if (message == "quit")
              break;
  
          // 当client首次发送消息给服务器的时候，OS会自动给client bind他的ip和port
          sendto(sock, message.c_str(), message.size(), 0, (struct sockaddr *)&server, sizeof(server));
  
          struct sockaddr_in temp;
          socklen_t len = sizeof(temp);
  
          // 读取数据
          ssize_t s = recvfrom(sock, buffer, sizeof(buffer), 0, (struct sockaddr *)&temp, &len);
          if (s > 0)
          {
              buffer[s] = 0;
              std::cout << "server echo# " << buffer << std::endl;
          }
          cout << 1 << endl;
      }
      close(sock);
  
      return 0;
  }
  ```

  

- udp_server.cc

  ```c++
  
  #include "udp_server.hpp"
  #include<iostream>
  #include<memory>
  
  static void usage(std::string proc)
  {
      std::cout << "\nUsage: " << proc << " ip" << " port\n" << std::endl;
  }
  
  
  // ./udp_server ip port 
  int main(int argc,char* argv[])
  {
      if(argc != 3)
      {
          usage(argv[0]);
          exit(1);
      }
  
      std::string ip = argv[1];
      uint16_t port = atoi(argv[2]);
      std::unique_ptr<UdpServer> svr(new UdpServer(port,ip));
      svr->initServer();
      svr->Start();
      return 0;
  }
  ```

  

- udp_server.hpp

  ```c++
  // 防止头文件的多重包含
  //  #ifndef 表示 "if not defined"，即如果尚未定义指定的宏，则执行后续操作。
  //  #define _UDP_SERVER_HPP 表示如果该宏未定义，则定义它，因此后续代码将会被包含在条件指令 #ifndef 和 #endif 之间。
  #ifndef _UDP_SERVER_HPP
  #define _UDP_SERVER_HPP
  
  #include "log.hpp"
  #include <string>
  #include <iostream>
  #include <cerrno>
  #include <unistd.h>
  #include <cstring>
  #include <stdio.h>
  
  // 网络四件套
  #include <sys/socket.h>
  #include <sys/types.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  
  class UdpServer
  {
  public:
      // 因为是文件描述符，所以没有初始化为-1
      UdpServer(uint16_t port, std::string ip = "0.0.0.0")
          : _port(port), _ip(ip), _sock(-1)
      {
      }
  
      // 从这里开始，就是新的调用，初始化服务器
      bool initServer()
      {
          // 1. 创造套接字
          // 网络通信就设置为  AF_INET  （IPv4）。
          // SOCK_DGRAM （套接字、用户数据报套接字）。
          _sock = socket(AF_INET, SOCK_DGRAM, 0);
          if (_sock < 0)
          {
              logMessage(FATAL, "%d : %s", errno, strerror(errno));
              exit(2);
          }
  
          // 2. bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
          struct sockaddr_in local;     // 是一个结构体，所以一般在使用的时候需要进行清零
          bzero(&local, sizeof(local)); // 将数据全部清零
          local.sin_family = AF_INET;   // 即：用来表示是，本地通讯还是网络通讯
  
          // 因为：服务器的IP和端口未来也是要发送给对方主机的（就如同打电话，是需要知道对方的电话号码）
          // -> 也就代表需要先将数据发送到网络 -> 代表需要注意大小端问题
          local.sin_port = htons(_port); // 端口号（2字节）- 主机序列转为网络序列
  
          // 1. 先要将点分十进制字符串风格的IP地址 -> 4字节
          // 2. 4字节主机序列 -> 网络序列
          // 有一套接口，可以一次帮我们做完这两件事情, 让服务器在工作过程中，可以从任意IP中获取数据:inet_addr()
                local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());
  
          if (bind(_sock, (struct sockaddr *)&local, sizeof(local)) < 0)
          {
              logMessage(FATAL, "%d : %s", errno, strerror(errno));
              exit(2);
          }
  
          logMessage(NORMAL, " init udp server done ... %s", strerror(errno));
  
          return true;
      }
  
      // 服务器开始运行
      void Start()
      {
          // 作为一款网络服务器，永远不退出的！
          // 服务器启动-> 进程 -> 常驻进程 -> 永远在内存中存在，除非挂了！
          // echo server: client给我们发送消息，我们原封不动返回（如同echo指令）
  
          while (true)
          {
              char buff[1024];
              // 注意：
              // peer：纯输出型参数
              struct sockaddr_in peer;
              bzero(&peer, sizeof(peer));
  
              // peer：输入输出型参数
              // 输入: peer 缓冲区大小
              // 输出: 实际读到的peer大小
              socklen_t len = sizeof(peer);
  
              // start.读取数据
              ssize_t s = recvfrom(_sock, buff, sizeof(buff) - 1, 0, (struct sockaddr *)&peer, &len);
              std::cout << "reuning" << std::endl;
              char result[256];
              std::string cmd_echo;
              if (s > 0)
              {
  
                  buff[s] = '\0'; // 将数据当作字符串
  
                  // 简易的防止，发送过来的字符串是指令：rm -rm ~
                  if (strcasestr(buff, "rm") != nullptr || strcasestr(buff, "rmdir") != nullptr)
                  {
                      std::string err_message = "乱删.... ";
                      std::cout << err_message << buff << std::endl;
                      sendto(_sock, err_message.c_str(), err_message.size(), 0, (struct sockaddr *)&peer, len);
                      continue;
                  }
                  // 希望执行：将发送过来的字符串（指令，如：ls -l -a），执行并将结果推回
                  FILE *fp = popen(buff, "r");
                  if (nullptr == fp)
                  {
                      logMessage(ERROR, "popen: %d:%s", errno, strerror(errno));
                      continue;
                  }
                  while (fgets(result, sizeof(result), fp) != nullptr) // 按行进行，从流当中将数据读取到缓冲区当中
                  {
                      cmd_echo += result;
                  }
                  fclose(fp);
                  // 分析和处理数据（忽略）
                  // end.写回数据
                  // sendto(_sock, buffer, strlen(buffer), 0, (struct sockaddr *)&peer, len);
              }
              sendto(_sock, cmd_echo.c_str(), cmd_echo.size(), 0, (struct sockaddr *)&peer, len);
          }
      }
  
      ~UdpServer()
      {
          if (_sock >= 0)
              close(_sock);
      }
  
  private:
      // 一个服务器，一般必须需要ip地址和port(16位的整数)
      uint16_t _port;  // 端口号
      std::string _ip; // ip地址
      int _sock;       // 文件描述符 - 套接字创建成功返回一个文件描述符
  };
  
  #endif
  
  ```

  

![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-18_23-56-46.gif)

![image-20240218235734574](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240218235734574.png)



---





## 9. **进行群聊天**（第三版）

（在于start运行的更改）**服务器的实现：**

此处想实现一个，服务器接收到客户端的数据后，将该用户的ip、端口、传输数据提取出来并存储，随后再将数据传送回。



- server.hpp

  > ```c++
  > // 防止头文件的多重包含
  > //  #ifndef 表示 "if not defined"，即如果尚未定义指定的宏，则执行后续操作。
  > //  #define _UDP_SERVER_HPP 表示如果该宏未定义，则定义它，因此后续代码将会被包含在条件指令 #ifndef 和 #endif 之间。
  > #ifndef _UDP_SERVER_HPP
  > #define _UDP_SERVER_HPP
  > 
  > #include "log.hpp"
  > #include <string>
  > #include <iostream>
  > #include <cerrno>
  > #include <unistd.h>
  > #include <cstring>
  > #include <stdio.h>
  > #include <queue>
  > #include <unordered_map>
  > 
  > // 网络四件套
  > #include <sys/socket.h>
  > #include <sys/types.h>
  > #include <netinet/in.h>
  > #include <arpa/inet.h>
  > 
  > class UdpServer
  > {
  > public:
  >     // 因为是文件描述符，所以没有初始化为-1
  >     UdpServer(uint16_t port, std::string ip = "0.0.0.0")
  >         : _port(port), _ip(ip), _sock(-1)
  >     {
  >     }
  > 
  >     // 从这里开始，就是新的调用，初始化服务器
  >     bool initServer()
  >     {
  >         // 1. 创造套接字
  >         // 网络通信就设置为  AF_INET  （IPv4）。
  >         // SOCK_DGRAM （套接字、用户数据报套接字）。
  >         _sock = socket(AF_INET, SOCK_DGRAM, 0);
  >         if (_sock < 0)
  >         {
  >             logMessage(FATAL, "%d : %s", errno, strerror(errno));
  >             exit(2);
  >         }
  > 
  >         // 2. bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
  >         struct sockaddr_in local;     // 是一个结构体，所以一般在使用的时候需要进行清零
  >         bzero(&local, sizeof(local)); // 将数据全部清零
  >         local.sin_family = AF_INET;   // 即：用来表示是，本地通讯还是网络通讯
  > 
  >         // 因为：服务器的IP和端口未来也是要发送给对方主机的（就如同打电话，是需要知道对方的电话号码）
  >         // -> 也就代表需要先将数据发送到网络 -> 代表需要注意大小端问题
  >         local.sin_port = htons(_port); // 端口号（2字节）- 主机序列转为网络序列
  > 
  >         // 1. 先要将点分十进制字符串风格的IP地址 -> 4字节
  >         // 2. 4字节主机序列 -> 网络序列
  >         // 有一套接口，可以一次帮我们做完这两件事情, 让服务器在工作过程中，可以从任意IP中获取数据:inet_addr()
  >         local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());
  > 
  >         if (bind(_sock, (struct sockaddr *)&local, sizeof(local)) < 0)
  >         {
  >             logMessage(FATAL, "%d : %s", errno, strerror(errno));
  >             exit(2);
  >         }
  > 
  >         logMessage(NORMAL, " init udp server done ... %s", strerror(errno));
  > 
  >         return true;
  >     }
  > 
  >     // 服务器开始运行
  >     void Start()
  >     {
  >         // 作为一款网络服务器，永远不退出的！
  >         // 服务器启动-> 进程 -> 常驻进程 -> 永远在内存中存在，除非挂了！
  >         // echo server: client给我们发送消息，我们原封不动返回（如同echo指令）
  > 
  >         while (true)
  >         {
  >             char buff[1024];
  >             // 注意：
  >             // peer：纯输出型参数
  >             struct sockaddr_in peer;
  >             bzero(&peer, sizeof(peer));
  > 
  >             // peer：输入输出型参数
  >             // 输入: peer 缓冲区大小
  >             // 输出: 实际读到的peer大小
  >             socklen_t len = sizeof(peer);
  > 
  >             // start.读取数据
  >             ssize_t s = recvfrom(_sock, buff, sizeof(buff) - 1, 0, (struct sockaddr *)&peer, &len);
  >             std::cout << "reuning" << std::endl;
  >             char key[1024];
  >             std::string cmd_echo;
  >             if (s > 0)
  >             {
  > 
  >                 buff[s] = '\0'; // 将数据当作字符串
  > 
  >                 uint16_t cli_port = ntohs(peer.sin_port);      // 从网络中来的！
  >                 std::string cli_ip = inet_ntoa(peer.sin_addr); // 4字节的网络序列的IP -> 本主机的字符串风格的IP，方便显示
  >                 // printf("[%s:%d]# %s\n", cli_ip.c_str(), cli_port, buffer);
  >                 snprintf(key, sizeof(key), "%s - %u", cli_ip.c_str(), cli_port); // 如：127 .0.0.1 - 8080
  >                 logMessage(NORMAL, "key: %s", key);
  > 
  >                 auto it = _users.find(key);
  >                 if (it == _users.end())
  >                 {
  >                     logMessage(NORMAL, "add new adder: %s", key);
  >                     _users.insert({key, peer});
  >                 }
  >             }
  >             // 分析和处理数据（忽略）
  >             // end.写回数据
  >             // sendto(_sock, buffer, strlen(buffer), 0, (struct sockaddr *)&peer, len);
  >             for (auto &iter : _users)
  >             {
  >                 std::string sendMessage = key;
  >                 sendMessage += "# ";
  >                 sendMessage += buff; // 如：127.0.0.1-8080# 你好
  >                 logMessage(NORMAL, "push message to %s", iter.first.c_str());
  >                 sendto(_sock,sendMessage.c_str(),sendMessage.size(),0,(struct sockaddr*)&iter.second,sizeof(iter.second));
  >             }
  >         }
  >     }
  > 
  >     ~UdpServer()
  >     {
  >         if (_sock >= 0)
  >             close(_sock);
  >     }
  > 
  > private:
  >     // 一个服务器，一般必须需要ip地址和port(16位的整数)
  >     uint16_t _port;                                             // 端口号
  >     std::string _ip;                                            // ip地址
  >     int _sock;                                                  // 文件描述符 - 套接字创建成功返回一个文件描述符
  >     std::unordered_map<std::string, struct sockaddr_in> _users; // 传递的数据与对应的网络套接字
  >     std::queue<std::string> messageQueue;                       // 存储用户（ip）
  > };
  > 
  > #endif
  > 
  > ```

> ![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-19_19-43-54.gif)

> ![image-20240219194542443](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240219194542443.png)



  可是在我们，再建立一个客户端之后，我们会发现：

![image-20240219195949449](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240219195949449.png)



-    两个客户端都存储进入了服务器。但是作为一个群聊天的实现，第二个客户端发送的信息并未显示。

  > **对于前面的客户端的实现：**
  >
  > ```c++
  > #include <iostream>
  > #include <cstring>
  > #include <unistd.h>
  > // 网络四件套
  > #include <sys/socket.h>
  > #include <sys/types.h>
  > #include <netinet/in.h>
  > #include <arpa/inet.h>
  >  
  > static void usage(std::string proc)
  > {
  >     std::cout << "\nUsage: " << proc << " serverIp serverPort\n"
  >               << std::endl;
  > }
  >  
  > int main(int argc, char *argv[])
  > {
  >     if (argc != 3)
  >     {
  >         usage(argv[0]);
  >         exit(1);
  >     }
  >     int sock = socket(AF_INET, SOCK_DGRAM, 0);
  >     if (sock < 0)
  >     {
  >         std::cerr << "socket error" << std::endl;
  >         exit(2);
  >     }
  >  
  >     // 此处需要使用ip和端口所以是需要bind的。
  >     // 但是一般client不会显示的bind，即：程序员不会自己bind。
  >     // 因为如果程序员bind，就一定是手动bind：ip和端口。
  >     // 这样是可以跑，但是不敢这么办（不建议）。
  >     // 因为：client是一个客户端 -> 普通人下载安装启动使用的-> 如果程序员自己bind了：
  >     // client 一定bind了一个固定的ip和port，万一，其他的客户端提前占用了这个port呢？？（因为一个端口号只能一个使用）
  >     // 所以：client一般不需要显示的bind指定port，而是让OS自动随机选择 —— OS自动选择没有被占用的端口号
  >     std::string message;
  >     struct sockaddr_in server;
  >     memset(&server, 0, sizeof(server));
  >     server.sin_family = AF_INET;
  >     server.sin_port = htons(atoi(argv[2]));
  >     server.sin_addr.s_addr = inet_addr(argv[1]);
  >  
  >     char buffer[1024];
  >     while (true)
  >     {
  >         std::cout << "请输入你的信息#";
  >         std::getline(std::cin, message);
  >         if(message == "quit") break;
  >         // 当client首次发送消息给服务器的时候，OS会自动给client bind他的ip和port
  >         sendto(sock, message.c_str(), message.size(), 0, (struct sockaddr *)&server, sizeof(server));
  >  
  >         struct sockaddr_in temp;
  >         socklen_t len = sizeof(temp);
  >  
  >         // 读取数据
  >         ssize_t s = recvfrom(sock, buffer, sizeof(buffer), 0, (struct sockaddr *)&temp, &len);
  >         if (s > 0)
  >         {
  >             buffer[s] = 0;
  >             std::cout << "server echo# " << buffer << std::endl;
  >         }
  >     }
  >     close(sock);
  >  
  >     return 0;
  > }
  > ```
  >
  >  上述的客户端，是单线程下的，所以只能先进行数据的发送再进行数据的读取。在此场景下就些许不合理了，应该是多线程的边进行数据的发送，边进行数据的读取。（所以此处我们需要改成多线程）
  >
  > 也就是关键问题，其实它是能够接收数据的，只不过其IO被阻塞了（接着第一个客户端中输入信息，就会将第二个客户端阻塞的数据输出），就是因为单线程。
  >
  > 
  >
  > ![img](https://img-blog.csdnimg.cn/84434340c8c44d6c94e1596d0130030a.png)

- **多线程客户端的实现：**

    实现一个线程发数据，一个线程收数据。



- 线程的封装

  > ```c++
  > #pragma once
  > #include <string>
  > #include <pthread.h>
  > #include <cstdio>
  >  
  > // 对线程的封装 - 不是完全必要，但是这样便于后期的统一管理
  >  
  > typedef void*(*fun_t)(void*);
  >  
  > // 整合线程的数据
  > class ThreadData
  > {
  > public:
  >     std::string _name;
  >     void* _args;
  > };
  >  
  > class Thread
  > {
  > public:
  >     Thread(int num, fun_t callback, void* args):_func(callback)
  >     {
  >         char nameBuffer[64];
  >         snprintf(nameBuffer, sizeof(nameBuffer), "Thread-%d", num);
  >         _name = nameBuffer;
  >  
  >         _tdata._args = args;
  >         _tdata._name = _name;
  >     }
  >  
  >     void start()
  >     {
  >         pthread_create(&_tid, nullptr, _func, (void*)&_tdata);
  >     }
  >  
  >     void join()
  >     {
  >         pthread_join(_tid, nullptr);
  >     }
  >  
  >     std::string name()
  >     {
  >         return _name;
  >     }
  >     
  >     ~Thread()
  >     {}
  >  
  > private:
  >    std::string _name;
  >    fun_t _func;
  >    ThreadData _tdata;
  >    pthread_t _tid;
  > };
  > ```





- 客互端

> ```c++
> 
> #include <iostream>
> #include <cstring>
> #include <unistd.h>
> #include <memory>
> #include "thread.hpp"
> 
> // 网络四件套
> #include <sys/socket.h>
> #include <sys/types.h>
> #include <netinet/in.h>
> #include <arpa/inet.h>
> 
> using namespace std;
> 
> uint16_t serverport = 0;
> std::string serverip;
> 
> static void usage(std::string proc)
> {
>     std::cout << "\nUsage: " << proc << " serverIp serverPort\n"
>               << std::endl;
> }
> 
> static void *udpSend(void *args)
> {
>     int sock = *(int *)((ThreadData *)args)->_args;
>     std::string name = ((ThreadData *)args)->_name;
>     std::string message;
>     struct sockaddr_in server;
>     memset(&server, 0, sizeof(server));
>     server.sin_family = AF_INET;
>     server.sin_port = htons(serverport);
>     server.sin_addr.s_addr = inet_addr(serverip.c_str());
> 
>     while(true)
>     {
>         while(true)
>         {
>              std::cout << "请输入你的信息#";
>             std::getline(std::cin, message);
>             if (message == "quit")
>                 break;
>             // 当client首次发送消息给服务器的时候，OS会自动给client bind他的ip和port
>             sendto(sock, message.c_str(), message.size(), 0, (struct sockaddr *)&server, sizeof(server));
>         }
>     }
>     return nullptr;
> }
> 
> static void *udpRecv(void *args)
> {
>     int sock = *(int *)((ThreadData *)args)->_args;
>     std::string name = ((ThreadData *)args)->_name;
> 
>     char buffer[1024];
> 
>     while (true)
>     {
>         memset(buffer, 0, sizeof(buffer));
>         struct sockaddr_in temp;
>         socklen_t len = sizeof(temp);
> 
>         // 读取数据
>         ssize_t s = recvfrom(sock, buffer, sizeof(buffer), 0, (struct sockaddr *)&temp, &len);
>         if (s > 0)
>         {
>             buffer[s] = 0;
>             std::cout << "server echo# " << buffer << std::endl;
>         }
>     }
>     return nullptr;
> }
> 
> // ./udp_client ip port
> int main(int argc, char *argv[])
> {
>     if (argc != 3)
>     {
>         usage(argv[0]);
>         exit(1);
>     }
> 
>     int sock = socket(AF_INET, SOCK_DGRAM, 0);
>     if (sock < 0)
>     {
>         std::cerr << "socket error" << std::endl;
>         exit(2);
>     }
> 
>     serverport = atoi(argv[2]);
>     serverip = argv[1];
> 
>     // 此处需要使用ip和端口所以是需要bind的。
>     // 但是一般client不会显示的bind，即：程序员不会自己bind。
>     // 因为如果程序员bind，就一定是手动bind：ip和端口。
>     // 这样是可以跑，但是不敢这么办（不建议）。
>     // 因为：client是一个客户端 -> 普通人下载安装启动使用的-> 如果程序员自己bind了：
>     // client 一定bind了一个固定的ip和port，万一，其他的客户端提前占用了这个port呢？？（因为一个端口号只能一个使用）
>     // 所以：client一般不需要显示的bind指定port，而是让OS自动随机选择 —— OS自动选择没有被占用的端口号
> 
>     std::unique_ptr<Thread> sender(new Thread(1, udpSend, (void *)&sock));
>     std::unique_ptr<Thread> recver(new Thread(1, udpRecv, (void *)&sock));
> 
>     sender->start();
>     recver->start();
> 
>     sender->join();
>     recver->join();
> 
>     close(sock);
>     return 0;
> }
> ```





![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-19_21-13-34.gif)



> 无论是多线程读还是写，用的sock都是一个，sock代表就是文件。```UDP是全双工的``` -> 可以同时进行收发而不受干扰。



- **全双工：**

> 全双工也就意味着，在发的时候同时也在收。

- **半双工：**

> 半双工在任意时刻只允许一个人进行收 / 发。 







## 10. windows 与linux 通讯（第四版）



- windows 下进行客互端传送
- linux 下进行服务器接收



windwos 客户端代码

```c++
#define _CRT_SECURE_NO_WARNINGS 1


#pragma warning(disable:4996)
#include <WinSock2.h>
#include <iostream>
#include <string>

using namespace std;
#pragma comment(lib,"ws2_32.lib") //固定用法

uint16_t serverport = 8080;
std::string serverip = "123.207.8.23";

int main()
{
	// windows 独有的
	WSADATA WSAData;
	WORD sockVersion = MAKEWORD(2, 2);
	if (WSAStartup(sockVersion, &WSAData) != 0)
		return 0;

	SOCKET clientSocket = socket(AF_INET, SOCK_DGRAM, 0);
	if (INVALID_SOCKET == clientSocket)
	{
		cout << "socket error!";
		return 0;
	}

	sockaddr_in dstAddr;
	dstAddr.sin_family = AF_INET;
	dstAddr.sin_port = htons(serverport);
	dstAddr.sin_addr.S_un.S_addr = inet_addr(serverip.c_str());

	char buffer[1024];

	while (true)
	{
		std::string message;
		std::cout << "请输入# ";
		std::getline(std::cin, message);
		sendto(clientSocket, message.c_str(), (int)message.size(), 0, (sockaddr*)&dstAddr, sizeof(dstAddr));

		struct sockaddr_in temp;
		int len = sizeof(temp);
		int s = recvfrom(clientSocket, buffer, sizeof buffer, 0, (sockaddr*)&temp, &len);
		if (s > 0)
		{
			buffer[s] = '\0';
			std::cout << "server echo# " << buffer << std::endl;
		}
	}

	// windows 独有
	closesocket(clientSocket);
	WSACleanup();

	return 0;
}
```



![image-20240220145348913](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240220145348913.png)













- 