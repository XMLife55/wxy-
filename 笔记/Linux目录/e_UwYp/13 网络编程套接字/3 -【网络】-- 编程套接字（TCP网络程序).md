# 1. TCP网络程序



## 1.1 **一个大致的模板**



- **log.hpp**

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



- **tcp_server.hpp**

```
#pragma once

#include<iostream>
#include "log.hpp"


class Tcp_server
{
public:
    Tcp_server(const uint16_t port,const std::string ip = "")
        :_port(port)
        ,_ip(ip)
    {}


    void initServer()
    {
        
    }

    void Start()
    {
        
    }
    
    ~Tcp_server()
    {
        
    }
private:
    uint16_t _port;
    std::string _ip;
    int _sock;
};
```



- tcp_server.cc

  ```
  
  #include"tcp_server.hpp"
  #include<memory>
  
  
  static void usage(std::string proc)
  {
      std::cout << "\nusage" << proc << "port\n" << std::endl;
  }
  
  int main(int argc,char* argv[])
  {
      if(argc != 2)
      {
          usage(argv[0]);
          exit(1);
      }
      uint16_t port = atoi(argv[1]);
      std::unique_ptr<Tcp_server> svr(new Tcp_server(port));
      svr->initServer();
      svr->Start();
      return 0;
  }
  ```

  

  ---



## 1.2 服务器前期知识储备



#### **socket**

-  与UD网络模型相同，TCP网络模型也需要使用到socket。

![img](https://img-blog.csdnimg.cn/2c6c179aebfe48859493c104b7ddb1f1.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// 创建套接字
int socket(int domain, int type, int protocol);
```

 socket属于计算机网络，给我们提供的一个系统调用接口，其是对传输层做了相关的一层文件系统级别的封装的一个接口。 



由于我们需要使用的是TCP，所以不同于UDP的```SOCK_DGRAM```，```而是SOCK_STREAM```。

- **SOCK_DGRAM**  - UDP

  > ![img](https://img-blog.csdnimg.cn/c3c63d157a2242dda8e3a755295ae969.png)

- **SOCK_STREAM** -  TCP

  > ![img](https://img-blog.csdnimg.cn/00a8b4ced8284f2e833c24f552211857.png)



- **返回值：**

>  套接字创建成功返回一个**文件描述符**，创建失败返回-1，同时错误码会被设置。



- **#问：**那后续的网络的读写是否可以采用，以前的文件接口来进行操作？

>  是这样的，在TCP协议中，套接字创建好，就与文件操作一摸一样。所以，socket函数接口的返回值，当成一个**套接字 / 文件描述符**就可以了。



- **参数说明：**

> **domain：**通常表示的是套接字的**域**（我们将来创建的套接字，是哪一种类型的套接字），也就是创建套接字的类型。
>
> ![img](https://img-blog.csdnimg.cn/2d94f83f779247aa969b7ae0597c788e.png)
>
>  其中我们最常用的就是：```AF_INET、AF_UNIX、 AF_LOCAL。```其中参数说白了就是宏，就相当于struct sockaddr结构的前16个位。如果是本地通信就设置为 ```AF_UNIX、AF_LOCAL ```，如果是网络通信就设置为 ```AF_INET （IPv4）。```

> **type：**类型，创建的套接字的通讯种类是什么。 
>
>  上述就是： ```SOCK_STREAM``` （套接字、用户数据报套接字）。

> protocol：只要其前面的两个参数是什么、怎么填已经确定了，它的协议也就基本上规定好了。它是用于创建套接字的协议类别，我们可以指明为TCP或UDP，但是其会根据传入的前两个参数自动推导出我们所需要使用的是哪种协议。所以，该字段一般直接设置为0就可以了，设置为0表示的就是默认。 

- **#问：**第一个参数与第二个参数有什么区别？

> **第一个参数：**说明了我们当前的套接字是用来进行**网络通讯**还是**本地通讯**的。
>
> **第二个参数：**如果确定是网络通讯了，那么想在网络当中**以什么方式进行通讯，**是以**数据流**还是**数据报**的方式。

---

#### **bind**

![img](https://img-blog.csdnimg.cn/e9d11240007f46458ea478aea439801a.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// bind绑定 - 将用户设置的ip和port在内核中和我们当前的进程强关联
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- **返回值：**

>  成功时，返回零。出现错误时，返回-1，并正确设置errno。



- **参数说明：**

>    **sockfd：**套接字对应的文件描述符。
>
> ​    **addr：**struct sockaddr_in类型的指针。
>
> ![img](https://img-blog.csdnimg.cn/21c15ef060904c1ca5964161d23f2ab1.png)
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
> ```
>
> ```c++
> // 通过宏，用##将符号拼接起来
> #define	__SOCKADDR_COMMON(sa_prefix) \
>   sa_family_t sa_prefix##family
> ```
>
> 对于网络地址（IP地址），比如："193.186.1.3"，称之为点分十进制风格的IP地址。由点作为分割符的每一个区域，在数字上取值范围是[0, 255]：1字节 -> 4个区域。理论上，表示一个IP地址，其实4字节就够了。4字节，每1个字节对应一个区域就行了。用字符串风格的显示，在网络通讯里没有必要，字符串风格是用于给用户看的。
>  于是需要：**点分十进制字符串风格的IP地址 <-> 4字节**。
>
> ```c++
> /* Internet address.  */
> typedef uint32_t in_addr_t;
> struct in_addr
>   {
>     in_addr_t s_addr;
>   };
> ```
>
> 我们需要定义一个struct sockaddr_in类型的对象，提供给bind。
>
> ```c++
> struct sockaddr_in local; // 是一个结构体，所以一般在使用的时候需要进行清零
> ```



- **Note：**

>    使用的时候，需要将头文件带齐。
>
> ```c++
> #include <netinet/in.h>
> #include <arpa/inet.h>
> ```
>
>  这两个头文件，会包含我们对应的所需要用的数据类型（如：struct sockaddr），工具方法。

---

#### **htonl、htons、ntohl、ntohs**

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

#### **inet_addr**

![img](https://img-blog.csdnimg.cn/13e52cac914a4ec1a153007267dc9ab1.png)

```c++
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
 
//将主机地址ip从IPv4数字和点符号转换为网络字节顺序的二进制数据。
in_addr_t inet_addr(const char *cp);
```

#### **listen**

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// 将套接字状态设置为监听状态
int listen(int sockfd, int backlog);
```

- **参数说明：**

>    **sockfd：**所创建好的套接字。
>
> ​    **backlog：**全连接队列的长度。

---

#### **accept**

 作为一款TCP服务器，其是面向连接的要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送），而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回 —— 函数**accept**。

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```



- **返回值：**

  > ​    如果成功，这些系统调用将返回一个非负整数，该整数是表示所接受套接字的描述符。出现错误时，将返回-1，并设置errno。 （说白了：返回值也就是一个套接字）

- **参数说明：**

> -   **sockfd：**所创建好的套接字。
>
> - **addr和**addrlen：输出型参数和输入输出型参数。
>
>   >    当有人过来的连接的时候，是想知道是谁向我发起的连接（客户端的ip、客户端的端口）。
>   >
>   >    
>   >
>   >    addr是一个传出参数,accept()返回时传出客户端的地址和端口号
>   >
>   >    如果给addr 参数传NULL,表示不关心客户端的地址



- accept的返回值的套接字与前面所创建的套接字有什么区别？

> **一个短故事：**一个餐厅有两类招待人的，一类门外，另一类是门内的。也就是说，当一群人从店门口路过，门外的服务人员就向他们推荐自家的店铺，并将客人引入店们。这个时候门内的服务人员就向前将客人接下，在餐桌上推销自家的菜品。而这个时候门外的服务人员就回去了，继续接引客人。
>
> - accept的套接字：门内的服务人员 —— 工作职责，通过 accept 获取上来新的连接，未来真正进行IO服务（网络服务）的不是 前面创建的套接字 ，而是 accept的套接字 。
>
>   ---
>
> - 前面创建的套接字：门外的服务人员 —— 工作职责，只是帮助 accept 把底层的连接获取上来。



TCP的读写IO可以运用两套接口，其中一套就是 read ， write 。



我们的服务器程序结构是这样的  

![image-20240222190836116](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240222190836116.png)

----

## 1.3 客互端前期知识储备

#### **connect**

![img](https://img-blog.csdnimg.cn/581d7d2d2ca7429887343f851d199385.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
// 在套接字上发起连接
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

> 客户端需要调用connect()连接服务器
>
> connect和bind的参数形式一致, 区别在于bind的参数是自己的地址, 而connect的参数是对方的地址; 



- **返回值：**

> 如果成功，0被返回。否者错误时，将返回-1，并设置errno。

- **参数说明：**

>  **sockfd：**所创建好的套接字。
>
>   **addr，addrlen：**代表将连哪个服务器。
>
>   connect作为系统调用接口，其内部会自动的给当前客户端绑定 服务端的ip和端口port。

---

**发送数据与读取数据的第二套接口：**

#### **send**

![img](https://img-blog.csdnimg.cn/f39752e2af8e4710b9b98bc281c9c626.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

 send 其是基于对应的TCP来向目标服务器发送消息，并且其前三个参数与返回值，是与write是一摸一样的，无非就是多了一个参数flags，并且一般还是设置为0。

---

#### **recv**

![img](https://img-blog.csdnimg.cn/900b78b2f84b44cb9c66535681418338.png)

```c++
#include <sys/types.h>
#include <sys/socket.h>
 
ssize_t recv(int sockfd, const void *buf, size_t len, int flags);
```

 ***recv**其是基于对应的TCP来接收消息，并且其前三个参数与返回值，是与write是一摸一样的，无非就是多了一个参数flags，并且一般还是设置为0。

---

#### 客户端代码

```c++


#include <iostream>
#include <unistd.h>
#include <string>
#include <cstdio>
#include <cstring>

// 网络四件套
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

void usage(std::string proc)
{
    std::cout << "\nUsage: " << proc << " serverIp serverPort\n"
              << std::endl;
}
// ./tcp_client targetIp targetPort
int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        usage(argv[0]);
        exit(1);
    }

    std::string serverid = argv[1];
    uint16_t port = atoi(argv[2]);
    int sock = 0;
    bool alive = false; // 确保连接没有才连接 - 确保连接状态
    std::string line;
    while (true)
    {
        if (!alive)
        {
            sock = socket(AF_INET, SOCK_STREAM, 0);
            if (sock < 0)
            {
                std::cerr << "socket error" << std::endl;
                exit(2);
            }

            // 客户端不需要进行bind
            // 因为作为客户端，其他应用也可就能是一个客户端
            // 如若bind一定意味着当前客户端进程绑定的一定是一个非常具体的端口号
            // 如果不同的客户端，端口号相撞就会出现问题

            // 需要操作系统自动进行port选择 - 达到连接别人的能力
            struct sockaddr_in server;
            bzero(&server, sizeof(server));
            server.sin_family = AF_INET;
            server.sin_port = htons(port);
            server.sin_addr.s_addr = inet_addr(serverid.c_str());

            if (connect(sock, (struct sockaddr *)&server, sizeof(server)) < 0)
            {
                std::cerr << "connect error" << std::endl;
                exit(3); // TODO
            }
            std::cout << "connect success" << std::endl;
            alive = true;
        }

        std::cout << "请输入# ";
        std::getline(std::cin, line);
        if (line == "quit")
            break;

        ssize_t s = send(sock, line.c_str(), line.size(), 0);
        if (s > 0)
        {
            char buffer[1024];
            ssize_t s = recv(sock, buffer, sizeof(buffer) - 1, 0);
            if (s > 0)
            {
                buffer[s] = 0;
                std::cout << "server 回显# " << buffer << std::endl;
            }
            else if (s == 0)
            {
                alive = false;
                close(sock);
            }
            else
            {
                std::cout << "recv error" << std::endl;
                break;
            }
        }
        else
        {
            alive = false;
            close(sock);
        }
    }
    return 0;
}
```



















---

## 1.4 version - 1 (单进程)

1.   单进程循环版 -- 只能够进行一次处理一个客户端，处理完了一个，才能处理下一个



- tcp_server.cc

  ```c++
  
  #include"tcp_server.hpp"
  #include<memory>
  
  
  static void usage(std::string proc)
  {
      std::cout << "\nusage" << proc << "port\n" << std::endl;
  }
  
  int main(int argc,char* argv[])
  {
      if(argc != 2)
      {
          usage(argv[0]);
          exit(1);
      }
      uint16_t port = atoi(argv[1]);
      std::unique_ptr<Tcp_server> svr(new Tcp_server(port));
      svr->initServer();
      svr->Start();
      return 0;
  }
  ```

  

- tcp_server.hpp

  ```c++
  #pragma once
  
  #include <iostream>
  #include <string>
  #include <cstring>
  #include <unistd.h>
  #include <signal.h>
  #include <cassert>
  #include "log.hpp"
  
  // 网络四件套
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  
  static void service(int sock, const std::string &clientip, const uint16_t &clientport)
  {
      // echo server
      char buffer[1024];
      while (true)
      {
          // read && write 可以直接被使用！
          ssize_t s = read(sock, buffer, sizeof(buffer));
          if (s > 0)
          {
              buffer[s] = 0; // 将发过来的数据当做字符串
              std::cout << clientip << ":" << clientport << "# " << buffer << std::endl;
          }
          else if (s == 0)
          {
              logMessage(NORMAL, "%s:%d shutdown, me too!", clientip.c_str(), clientport);
              break;
          }
          else
          {
              logMessage(ERROR, "read socket error, %d:%s", errno, strerror(errno));
              break;
          }
  
          write(sock, buffer, strlen(buffer));
      }
  }
  
  class Tcp_server
  {
  private:
      const static int gbacklog = 20; // 一般不能太大也不能太小
  public:
      Tcp_server(const uint16_t port, const std::string ip = "")
          : _port(port), _ip(ip), _listensock(-1)
      {
      }
  
      void initServer()
      {
          // 1. 创建socket
          _listensock = socket(AF_INET, SOCK_STREAM, 0);
          if (_listensock < 0)
          {
              logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
              exit(2);
          }
          logMessage(NORMAL, "create socket success, _sock: %d", _listensock); // 验证其是3: 因为底层文件描述符表3没给占用
  
          // 2. bind绑定ip和端口号
          struct sockaddr_in local;
          bzero(&local, sizeof(local));
          local.sin_family = AF_INET;
          local.sin_port = htons(_port);
          local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());
  
          if (bind(_listensock, (struct sockaddr *)&local, sizeof(local)) < 0)
          {
              logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
              exit(3);
          }
  
          // 3. 因为TCP是面向连接的， 当我们正式进行通讯的时候，需要先建立连接
          // 而需要以连接进行通讯，也就代表了需要进行等待连接成功
          if (listen(_listensock, gbacklog) < 0)
          {
              logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
              exit(4);
          }
  
          logMessage(NORMAL, "create server success");
      }
  
      void Start()
      {
          while (1)
          {
              // 4. 获取连接
              struct sockaddr_in src;
              socklen_t len = sizeof(src);
              // 作为一款TCP服务器，其是面向连接的
              // 要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送）
              // 而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回
              // （即：函数accept）
              int servicesock = accept(_listensock, (struct sockaddr *)&src, &len);
              if (servicesock < 0)
              {
                  logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
                  continue;
              }
  
              // 获取连接成功了
              uint16_t client_port = ntohs(src.sin_port);
              std::string client_ip = inet_ntoa(src.sin_addr);
              logMessage(NORMAL, "link success, servicesock: %d | %s : %d |\n",
                         servicesock, client_ip.c_str(), client_port);
  
              // 开始正常的通讯服务
              // version 1 -- 单进程循环版 -- 只能够进行一次处理一个客户端，处理完了一个，才能处理下一个
              service(servicesock, client_ip, client_port);
              close(servicesock);
          }
      }
  
      ~Tcp_server()
      {
      }
  
  private:
      uint16_t _port;
      std::string _ip;
      int _listensock;
  };
  
  ```

  

#### telnet

> 一个个工具： telnet 远程登陆工具，可以让我们直接输入对应的ip、port直接进行网络，使得此处无需自己写客户端，也可以直接运行测试。

```
（安装：sudo yum -y install telnet）
```



> - **（^] == Ctrl + ]）**
>
>   ![img](https://img-blog.csdnimg.cn/46f44cfb858748e4b139d75cdcfbd4d6.png)
>
>   **使用：**
>
>   ![img](https://img-blog.csdnimg.cn/9036a6ba6d9b44d88836d995cf838cb0.png)
>
>   **退出：**
>
>   ![img](https://img-blog.csdnimg.cn/a9909b78bd8d40c98c85404c54932083.png)

![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-21_09-01-00.gif)



 此处我们所写的是一个**单进程**的，于是很显然单进程获取连接成功，然后进行service。并且这个service内部可是一个死循环，换句话说，其作为单进程进入service，就会一直读写，如果不退出，就无法回到之前的accept继续获取连接继续处理。



- **其一的解决方式就是利用**多进程

> - 父进程继续accept获取新连接。
> - 让子进程给客户端提供服务。

---



## 1.5 version - 2 (多进程)



利用多进程解决 无法 连续 accept继续获取连接继续处理数据。

只需在 tcp_server.hpp 中的 start 中构造即可



- tcp_server.hpp

  > - SIGCHLD 信号是在子进程结束时由内核发送给父进程的信号，通知父进程子进程的状态变化。
  >
  > - `signal(SIGCHLD, SIG_IGN)` 表示将 SIGCHLD 信号的处理方式设置为忽略，即当父进程收到 SIGCHLD 信号时，不进行任何处理，由系统自动回收子进程资源，避免产生僵尸进程。

  ```c++
  #pragma once
  
  #include <iostream>
  #include <string>
  #include <cstring>
  #include <unistd.h>
  #include <signal.h>
  #include <cassert>
  #include "log.hpp"
  
  // 网络四件套
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  
  static void service(int sock, const std::string &clientip, const uint16_t &clientport)
  {
      // echo server
      char buffer[1024];
      while (true)
      {
          // read && write 可以直接被使用！
          ssize_t s = read(sock, buffer, sizeof(buffer));
          if (s > 0)
          {
              buffer[s] = 0; // 将发过来的数据当做字符串
              std::cout << clientip << ":" << clientport << "# " << buffer << std::endl;
          }
          else if (s == 0)
          {
              logMessage(NORMAL, "%s:%d shutdown, me too!", clientip.c_str(), clientport);
              break;
          }
          else
          {
              logMessage(ERROR, "read socket error, %d:%s", errno, strerror(errno));
              break;
          }
  
          write(sock, buffer, strlen(buffer));
      }
  }
  
  class Tcp_server
  {
  private:
      const static int gbacklog = 20; // 一般不能太大也不能太小
  public:
      Tcp_server(const uint16_t port, const std::string ip = "")
          : _port(port), _ip(ip), _listensock(-1)
      {
      }
  
      void initServer()
      {
          // 1. 创建socket
          _listensock = socket(AF_INET, SOCK_STREAM, 0);
          if (_listensock < 0)
          {
              logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
              exit(2);
          }
          logMessage(NORMAL, "create socket success, _sock: %d", _listensock); // 验证其是3: 因为底层文件描述符表3没给占用
  
          // 2. bind绑定ip和端口号
          struct sockaddr_in local;
          bzero(&local, sizeof(local));
          local.sin_family = AF_INET;
          local.sin_port = htons(_port);
          local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());
  
          if (bind(_listensock, (struct sockaddr *)&local, sizeof(local)) < 0)
          {
              logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
              exit(3);
          }
  
          // 3. 因为TCP是面向连接的， 当我们正式进行通讯的时候，需要先建立连接
          // 而需要以连接进行通讯，也就代表了需要进行等待连接成功
          if (listen(_listensock, gbacklog) < 0)
          {
              logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
              exit(4);
          }
  
          logMessage(NORMAL, "create server success");
      }
  
      void Start()
      {
          signal(SIGCHLD, SIG_IGN); // 对SIGCHLD，主动忽略SIGCHLD信号，子进程退出的时候，会自动释放自己的僵尸状态
          while (1)
          {
              // 4. 获取连接
              struct sockaddr_in src;
              socklen_t len = sizeof(src);
              // 作为一款TCP服务器，其是面向连接的
              // 要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送）
              // 而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回
              // （即：函数accept）
              int servicesock = accept(_listensock, (struct sockaddr *)&src, &len);
              if (servicesock < 0)
              {
                  logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
                  continue;
              }
  
              // 获取连接成功了
              uint16_t client_port = ntohs(src.sin_port);
              std::string client_ip = inet_ntoa(src.sin_addr);
              logMessage(NORMAL, "link success, servicesock: %d | %s : %d |\n",
                         servicesock, client_ip.c_str(), client_port);
  
              // 开始正常的通讯服务
              // version 2.0 -- 多进程版 --- 创建子进程
              // 让子进程给新的连接提供服务，子进程能不能打开父进程曾经打开的文件fd呢？
              pid_t id = fork();
              if (id == 0)
              {
                  // 子进程， 子进程会继承父进程打开的文件与文件fd。
                  // 让子进程给客户端提供服务
                  // 让父进程继续accept获取新连接
                  
                  close(_listensock); // 子进程是来进行提供服务的，发生写时拷贝此时是不需要监听socket的
                  service(servicesock, client_ip, client_port);
                  close(servicesock); //不写也可以 因为局部变量退出时会自动释放关闭
                  exit(0); // 会进入僵尸状态
              }
              // 父进程
              /*解决僵尸进程的方式*/
  
              // 1、阻塞等待
              // 并且我们无法使用waitpid()，因为其本身就是阻塞等待
  
              // 2、非阻塞等待
              // 虽然我们可以使用非阻塞等待，但是其是在是太恶心了。
              // 首先我们需要将所有子进程的pid保存起来，并且还需要不断循环式的遍历 - 太麻烦
  
              // 3、信号捕捉
              // 我们是可以通过子进程退出向入进程发信号的特点，进行信号的捕捉
              // 但是也不够好
  
              // 另外好的两种方法
              // 1. signal(SIGCHLD, SIG_IGN);
  
              // 另外好的两种方法
              // 1. signal(SIGCHLD, SIG_IGN);
              // 2. 见后面的进阶
              close(servicesock);
          }
      }
  
      ~Tcp_server()
      {
      }
  
  private:
      uint16_t _port;
      std::string _ip;
      int _listensock;
  };
  
  ```

  ![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-02-21_09-25-12.gif)



#### *version2.1* 进阶版

> `waitpid(id, nullptr, 0);` 的作用是等待特定的子进程结束，并获取其状态。
>
> 具体来说：
>
> - `waitpid` 函数是一个用于等待子进程结束的系统调用。
> - 参数 `id` 是要等待的子进程的进程ID。
> - 参数 `nullptr` 表示不关心子进程的状态信息。
> - 参数 `0` 表示等待任意状态的子进程，包括已终止的子进程、被暂停的子进程等
>
> 阻塞当前进程，直到指定的子进程结束。在这个例子中，等待进程ID为 `id` 的子进程结束，并忽略其状态信息，一旦子进程结束，`waitpid` 函数就会返回。这样可以避免产生僵尸进程，并确保子进程的资源被正确回收。

```c++
#pragma once

#include <iostream>
#include <string>
#include <cstring>
#include <unistd.h>
#include <signal.h>
#include <cassert>
#include "log.hpp"

// 网络四件套
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <arpa/inet.h>

static void service(int sock, const std::string &clientip, const uint16_t &clientport)
{
    // echo server
    char buffer[1024];
    while (true)
    {
        // read && write 可以直接被使用！
        ssize_t s = read(sock, buffer, sizeof(buffer));
        if (s > 0)
        {
            buffer[s] = 0; // 将发过来的数据当做字符串
            std::cout << clientip << ":" << clientport << "# " << buffer << std::endl;
        }
        else if (s == 0)
        {
            logMessage(NORMAL, "%s:%d shutdown, me too!", clientip.c_str(), clientport);
            break;
        }
        else
        {
            logMessage(ERROR, "read socket error, %d:%s", errno, strerror(errno));
            break;
        }

        write(sock, buffer, strlen(buffer));
    }
}

class Tcp_server
{
private:
    const static int gbacklog = 20; // 一般不能太大也不能太小
public:
    Tcp_server(const uint16_t port, const std::string ip = "")
        : _port(port), _ip(ip), _listensock(-1)
    {
    }

    void initServer()
    {
        // 1. 创建socket
        _listensock = socket(AF_INET, SOCK_STREAM, 0);
        if (_listensock < 0)
        {
            logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
            exit(2);
        }
        logMessage(NORMAL, "create socket success, _sock: %d", _listensock); // 验证其是3: 因为底层文件描述符表3没给占用

        // 2. bind绑定ip和端口号
        struct sockaddr_in local;
        bzero(&local, sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());

        if (bind(_listensock, (struct sockaddr *)&local, sizeof(local)) < 0)
        {
            logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
            exit(3);
        }

        // 3. 因为TCP是面向连接的， 当我们正式进行通讯的时候，需要先建立连接
        // 而需要以连接进行通讯，也就代表了需要进行等待连接成功
        if (listen(_listensock, gbacklog) < 0)
        {
            logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
            exit(4);
        }

        logMessage(NORMAL, "create server success");
    }

    void Start()
    {
        signal(SIGCHLD, SIG_IGN); // 对SIGCHLD，主动忽略SIGCHLD信号，子进程退出的时候，会自动释放自己的僵尸状态
        while (1)
        {
            // 4. 获取连接
            struct sockaddr_in src;
            socklen_t len = sizeof(src);
            // 作为一款TCP服务器，其是面向连接的
            // 要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送）
            // 而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回
            // （即：函数accept）
            int servicesock = accept(_listensock, (struct sockaddr *)&src, &len);
            if (servicesock < 0)
            {
                logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
                continue;
            }

            // 获取连接成功了
            uint16_t client_port = ntohs(src.sin_port);
            std::string client_ip = inet_ntoa(src.sin_addr);
            logMessage(NORMAL, "link success, servicesock: %d | %s : %d |\n",
                       servicesock, client_ip.c_str(), client_port);

            // 开始正常的通讯服务
            // version2.1 -- 多进程版
            pid_t id = fork();
            if (id == 0)
            {
                // 子进程
                close(_listensock);
                //提前让主进程的子进程退出 直接回收，不用阻塞！
                if (fork() > 0 /*子进程本身*/)
                    exit(0); // 子进程本身立即退出，让孙子进程执行后续
                             // 孙子进程变为孤儿进程，于是操作系统领养，操作系统在退出的时候，由操作系统自动回收孤儿进程！
                service(servicesock, client_ip, client_port);
                exit(0);
            }

            waitpid(id,nullptr,0); // 不会阻塞！
            close(servicesock);
        }
    }

    ~Tcp_server()
    {
    }

private:
    uint16_t _port;
    std::string _ip;
    int _listensock;
};

```



- 多进程解决的缺陷？

> 创建进程的成本太高了，其需要创建PCB、创建地址空间，创建页表结构、调度、为进程分配对应的资源（进程是承担操作系统资源的基本单位）……。



- 我们可以考虑改为多线程。

---

## 1.6 version - 3 (多线程)

- 只需在服务器的 ```server.hpp``` 改动即可

```c++
#pragma once

#include <iostream>
#include <string>
#include <cstring>
#include <unistd.h>
#include <signal.h>
#include <cassert>
#include <pthread.h>
#include "log.hpp"

// 网络四件套
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <arpa/inet.h>

static void service(int sock, const std::string &clientip, const uint16_t &clientport)
{
    // echo server
    char buffer[1024];
    while (true)
    {
        // read && write 可以直接被使用！
        ssize_t s = read(sock, buffer, sizeof(buffer));
        if (s > 0)
        {
            buffer[s] = 0; // 将发过来的数据当做字符串
            std::cout << clientip << ":" << clientport << "# " << buffer << std::endl;
        }
        else if (s == 0)
        {
            logMessage(NORMAL, "%s:%d shutdown, me too!", clientip.c_str(), clientport);
            break;
        }
        else
        {
            logMessage(ERROR, "read socket error, %d:%s", errno, strerror(errno));
            break;
        }

        write(sock, buffer, strlen(buffer));
    }
}


class ThreadData
{
public:
    int _sock;
    std::string ip;
    uint16_t port;
};

class Tcp_server
{
private:
    const static int gbacklog = 20; // 一般不能太大也不能太小
private:
    static void* threadRoutine(void* args)
    {
        pthread_detach(pthread_self()); //线程分离
        ThreadData* td = (ThreadData*)args;
        service(td->_sock,td->ip,td->port);
        close(td->_sock); //关闭accept的套接字
        delete td;
        return nullptr;
    }
public:
    Tcp_server(const uint16_t port, const std::string ip = "")
        : _port(port), _ip(ip), _listensock(-1)
    {
    }

    void initServer()
    {
        // 1. 创建socket
        _listensock = socket(AF_INET, SOCK_STREAM, 0);
        if (_listensock < 0)
        {
            logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
            exit(2);
        }
        logMessage(NORMAL, "create socket success, _sock: %d", _listensock); // 验证其是3: 因为底层文件描述符表3没给占用

        // 2. bind绑定ip和端口号
        struct sockaddr_in local;
        bzero(&local, sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());

        if (bind(_listensock, (struct sockaddr *)&local, sizeof(local)) < 0)
        {
            logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
            exit(3);
        }

        // 3. 因为TCP是面向连接的， 当我们正式进行通讯的时候，需要先建立连接
        // 而需要以连接进行通讯，也就代表了需要进行等待连接成功
        if (listen(_listensock, gbacklog) < 0)
        {
            logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
            exit(4);
        }

        logMessage(NORMAL, "create server success");
    }

    void Start()
    {
        signal(SIGCHLD, SIG_IGN); // 对SIGCHLD，主动忽略SIGCHLD信号，子进程退出的时候，会自动释放自己的僵尸状态
        while (1)
        {
            // 4. 获取连接
            struct sockaddr_in src;
            socklen_t len = sizeof(src);
            // 作为一款TCP服务器，其是面向连接的
            // 要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送）
            // 而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回
            // （即：函数accept）
            int servicesock = accept(_listensock, (struct sockaddr *)&src, &len);
            if (servicesock < 0)
            {
                logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
                continue;
            }

            // 获取连接成功了
            uint16_t client_port = ntohs(src.sin_port);
            std::string client_ip = inet_ntoa(src.sin_addr);
            logMessage(NORMAL, "link success, servicesock: %d | %s : %d |\n",
                       servicesock, client_ip.c_str(), client_port);

            // version 3 --- 多线程版本
            ThreadData* td = new ThreadData(); // 此处不要使用 ThreadData td; 因为这样是在栈上定义的对象，不是线程安全的。
            td->_sock = servicesock;
            td->port = client_port;
            td->ip = client_ip;
            pthread_t pid;
            pthread_create(&pid,nullptr,threadRoutine,td);
        }
    }

    ~Tcp_server()
    {
    }

private:
    uint16_t _port;
    std::string _ip;
    int _listensock;
};

```

  但是对于多线程，当任务来的时候，就需要创建线程，创建线程也是消耗。所以我们可以采取创建一个线程池，将线程先创建好，需要就拿一个线程进行执行。

---



## 1.7 version - 4(线程池）

**（此处：使用的是一个单例版本的线程池）**

- **lockGuard.hpp**

> 锁的封装

```c++
#pragma once

#include<iostream>
#include<pthread.h>


// RAII风格的加锁方式
class lockGuard
{
public:
    lockGuard(pthread_mutex_t* mtx)
        :_mtx(mtx)
    {
        pthread_mutex_init(_mtx,nullptr);
    }

    ~lockGuard()
    {
        pthread_mutex_destroy(_mtx);
    }
private:
    pthread_mutex_t * _mtx;
};
```



- Task.hpp

  > 任务的封装。

```c++
#pragma once

#include<iostream>
#include<string>
#include<functional>


// 两种书写方式 - 是等价的
//static void service(int sock, const std::string &clientip, const uint16_t &clientport,std::string& thread_name)

//typedef std::function<void(int, const std::string&, const uint16_t &,const std::string&)> func_t;
using func_t = std::function<void (int, const std::string &, const uint16_t &, const std::string &)>;
class Task
{
public:
    Task()
    {}
public:
    Task(int sock, const std::string ip,uint16_t port,func_t func)
        :_sock(sock)
        ,_ip(ip)
        ,_port(port)
        ,_func(func)
    {} 

    void operator()(const std::string& name)
    {
        _func(_sock,_ip,_port,name);
    }

private:
    int _sock;
    std::string _ip;
    uint16_t _port;
    func_t _func; //回调函数
};

```



- **thread.hpp**

>  线程的封装。

```c++
#pragma once
#include<string>
#include<pthread.h>
#include<cstdio>


// 对线程的封装 - 不是完全必要，但是这样便于后期的统一管理
typedef void*(*fun_t)(void*);


class ThreadData
{
public:
    std::string _name;
    void* _args;
};

class Thread
{
public:
    Thread(int num,fun_t callback,void* args)
        :_func(callback)
    {
        char nameBuffer[64];
        snprintf(nameBuffer,sizeof(nameBuffer),"Thread - %d",num); //显示线程几
        _name = nameBuffer;
        
        //线程中的数据
        _tdata._args = args;
        _tdata._name = nameBuffer;
    }

    //启动线程
    void start()
    {
        pthread_create(&_tid,nullptr,_func,(void*)&_tdata);
    }

    //等待线程
    void join()
    {
        pthread_join(_tid,nullptr);
    }

     // 未来不再使用线程id了，因为其是一个地址不便于我们查看
    std::string name()
    {
        return _name;
    }

    ~Thread()
    {}
    
private:
    std::string _name;
    fun_t _func;
    ThreadData _tdata;
    pthread_t _tid;
};
```



- **threadPool.hpp**

> 线程池的实现。

```c++
#include "thread.hpp"
#include "lockGuard.hpp"
#include <iostream>
#include <vector>
#include <queue>
#include <unistd.h>

const int g_thread_num = 10;

template <class T>
class ThreadPool
{
public:
    // 返回锁的地址
    pthread_mutex_t *getMutex()
    {
        return &_lock;
    }

    bool isEmpty()
    {
        return _task_queue.empty();
    }

    void waitCond()
    {
        pthread_cond_wait(&_cond, &_lock);
    }

    T getTask()
    {
        T task = _task_queue.front();
        _task_queue.pop();
        return task;
    }

public:
    // 1.run - 将线程跑起来
    void run()
    {
        int i = 0;
        for (auto &iter : _threads)
        {
            iter->start();
            // std::cout << iter->name() << " 启动成功 " << std::endl;
            logMessage(NORMAL, "%s %s", iter->name().c_str(), "启动成功");
        }
    }

    // 未来所有执行流所执行的方法 - 核心取任务、执行任务的逻辑
    static void *routine(void *args) // 因为在类当中，如果是一个成员方法，其会有一个隐藏的参数this指针，所以我们需要使用static进行修饰
    {
        ThreadData *td = (ThreadData *)args;
        ThreadPool<T> *tp = (ThreadPool<T> *)td->_args;
        while (true)
        {
            T task; // 对应任务
            // 利用{}确定加锁的区间
            {
                // lock
                lockGuard lockguard(tp->getMutex());

                // while(task_queue_.empty()) wait();
                while (tp->isEmpty())
                    tp->waitCond();

                //  获取任务 - 100%有任务
                task = tp->getTask(); // 任务队列是共享的->将任务从共享，拿到自己的私有空间

            }                // 自动释放锁
                             // 处理任务
            task(td->_name); // 要求每一个任务都要提供一个仿函数
        }
    }

    // 2.pushTask - 将任务放到任务池里
    void pushTask(const T &task)
    {
        lockGuard lockguard(&_lock); // 加锁
        _task_queue.push(task);      // 压入任务
        pthread_cond_signal(&_cond); // 发送信号唤醒线程
                                     // 自动释放锁
    }

    ~ThreadPool()
    {
        for (auto &iter : _threads)
        {
            iter->join();
            delete iter;
        }
        pthread_mutex_destroy(&_lock);
        pthread_cond_destroy(&_cond);
    }

public:
    // 提供一个函数创建对象
    // 想获取对象就只有调用这个函数
    static ThreadPool<T> *GetThreadPool(int num = g_thread_num)
    {
        //防止出现:大量的申请和释放锁的行为，而导致的无用且浪费资源的行为
        if (_pthread_ptr == nullptr)
        {
            lockGuard lockguard(&_mutex);
            // 由于_thread_ptr是由static修饰的 - 只有一份
            if (_pthread_ptr == nullptr)
            {
                _pthread_ptr = new ThreadPool<T>(num);
            }
        }
        return _pthread_ptr; // 返回的永远都是同一个线程池对象"
    }

private:
    ThreadPool(int thread_num = g_thread_num)
        : _num(thread_num)
    {
        for (int i = 1; i <= _num; i++)
        {
            _threads.push_back(new Thread(i, routine, this));
        }

        pthread_mutex_init(&_lock, nullptr);
        pthread_cond_init(&_cond, nullptr);
    }

    ThreadPool(const ThreadPool<T> &other) = delete;
    const ThreadPool<T> &operator=(const ThreadPool<T> &other) = delete;

    
private:
    std::vector<Thread *> _threads; // 存储线程
    int _num;                       // 线程个数
    std::queue<T> _task_queue;      // 线程任务
    pthread_mutex_t _lock;          // 锁
    pthread_cond_t _cond;           // 条件变量
    static ThreadPool<T> *_pthread_ptr;
    static pthread_mutex_t _mutex;
};

// 在类外来对静态成员进行初始化

template <class T>
ThreadPool<T> *ThreadPool<T>::_pthread_ptr = nullptr;

template <class T>
pthread_mutex_t ThreadPool<T>::_mutex = PTHREAD_MUTEX_INITIALIZER;
```



- server.hpp

  ```c++
  #pragma once
  
  #include <iostream>
  #include <string>
  #include <cstring>
  #include <unistd.h>
  #include <signal.h>
  #include <cassert>
  #include <pthread.h>
  #include <memory>
  #include <string>
  #include "log.hpp"
  #include "ThreadPool.hpp"
  #include "Task.hpp"
  
  // 网络四件套
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <sys/wait.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  
  static void service(int sock, const std::string &clientip, const uint16_t &clientport, const std::string &name)
  {
      // echo server
      char buffer[1024];
      while (true)
      {
          // read && write 可以直接被使用！
          ssize_t s = read(sock, buffer, sizeof(buffer));
          if (s > 0)
          {
              buffer[s] = 0; // 将发过来的数据当做字符串
              std::cout << name << " | " << clientip << ":" << clientport << "# " << buffer << std::endl;
          }
          else if (s == 0)
          {
              logMessage(NORMAL, "%s:%d shutdown, me too!", clientip.c_str(), clientport);
              break;
          }
          else
          {
              logMessage(ERROR, "read socket error, %d:%s", errno, strerror(errno));
              break;
          }
  
          write(sock, buffer, strlen(buffer));
      }
      close(sock);
  }
  
  class Tcp_server
  {
  private:
      const static int gbacklog = 20; // 一般不能太大也不能太小
  private:
  public:
      Tcp_server(const uint16_t port, const std::string ip = "0.0.0.0")
          : _port(port), _ip(ip), _listensock(-1), _threadpool_ptr(ThreadPool<Task>::GetThreadPool())
      {
      }
  
      void initServer()
      {
          // 1. 创建socket
          _listensock = socket(AF_INET, SOCK_STREAM, 0);
          if (_listensock < 0)
          {
              logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
              exit(2);
          }
          logMessage(NORMAL, "create socket success, _sock: %d", _listensock); // 验证其是3: 因为底层文件描述符表3没给占用
  
          // 2. bind绑定ip和端口号
          struct sockaddr_in local;
          bzero(&local, sizeof(local));
          local.sin_family = AF_INET;
          local.sin_port = htons(_port);
          local.sin_addr.s_addr = _ip.empty() ? INADDR_ANY : inet_addr(_ip.c_str());
  
          if (bind(_listensock, (struct sockaddr *)&local, sizeof(local)) < 0)
          {
              logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
              exit(3);
          }
  
          // 3. 因为TCP是面向连接的， 当我们正式进行通讯的时候，需要先建立连接
          // 而需要以连接进行通讯，也就代表了需要进行等待连接成功
          if (listen(_listensock, gbacklog) < 0)
          {
              logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
              exit(4);
          }
  
          logMessage(NORMAL, "create server success");
      }
  
      void Start()
      {
          _threadpool_ptr->run();
          while (1)
          {
              // 4. 获取连接
              struct sockaddr_in src;
              socklen_t len = sizeof(src);
              // 作为一款TCP服务器，其是面向连接的
              // 要正常通讯，别人需要先发起建立连接的请求（UDP是直接将数据发送）
              // 而TCP是需要先进行连接的获取，获取连接的前提就是有人进行连接，所以如果没有人连接，就一直阻塞等待，有人连接即直接返回
              // （即：函数accept）
              int servicesock = accept(_listensock, (struct sockaddr *)&src, &len);
              if (servicesock < 0)
              {
                  logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
                  continue;
              }
  
              // 获取连接成功了
              uint16_t client_port = ntohs(src.sin_port);
              std::string client_ip = inet_ntoa(src.sin_addr);
              logMessage(NORMAL, "link success, servicesock: %d | %s : %d |\n",
                         servicesock, client_ip.c_str(), client_port);
  
              // verison4 --- 线程池版本
              Task t(servicesock, client_ip, client_port, service);
              _threadpool_ptr->pushTask(t);
          }
      }
  
      ~Tcp_server()
      {
      }
  
  private:
      uint16_t _port;
      std::string _ip;
      int _listensock;
      std::unique_ptr<ThreadPool<Task>> _threadpool_ptr; 
  };
  
  ```

  



- server.cc

```c++

#include"tcp_server.hpp"
#include<memory>


static void usage(std::string proc)
{
    std::cout << "\nusage" << proc << "port\n" << std::endl;
}

int main(int argc,char* argv[])
{
    if(argc != 2)
    {
        usage(argv[0]);
        exit(1);
    }
    uint16_t port = atoi(argv[1]);
    std::unique_ptr<Tcp_server> svr(new Tcp_server(port));
    svr->initServer();
    svr->Start();
    return 0;
}
```



- **上述代码的问题：**

>  线程池是有一个固定数量n的线程的，如果已经有对应数目的客户端，申请完了线程池中的线程，那么后续的客户端只能将放在任务队列中等待执行。所以，一般服务器进程业务处理，如果是从连上到断开，要一直保持这个连接，其实是很少出现的（服务器也要避免如此的为他人长时间提供服务）。
>
> 服务起对于多进程和多线程一定要有明显的上限，不然一瞬间的大量请求就会导致服务器崩溃，这个也是线程池的优势。



> **端口号细节问题：**
>
> 
>
> 无论是客户端还是服务器，是都需要端口号的：
>
> - 服务器必须要是明确的端口号 —— 因为其面对众多客户端，一旦服务器端口号轻易的被更改了，所有客户端就无法连接服务器了。
> - 客户端需要的端口号，主要的是为了唯一性，具体明确的数值我们是完全不关心的。