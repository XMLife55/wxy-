```应用层：```就是程序员基于socket接口之上编写的具体逻辑，都是和文本处理有关的 -- 协议分析与处理。

```http协议```：具有大量的文本分析和协议处理。



## 1. **HTTP协议**

> 虽然我们说,， 应用层协议是我们程序猿自己定的。但实际上， 已经有大佬们定义了一些现成的， 又非常好用的应用层协议， 供我们直接参考使用 ->  HTTP （超文本传输协议） 就是其中之一。

---

## 2. 认识URL



URL 是统一资源定位符（Uniform Resource Locator）的缩写，用于标识互联网上资源的地址。它是一个全球性的标准，用于指定资源在互联网上的位置。



- 平时我们俗称的 " 网址 " 其实就是说的 URL。

![img](https://img-blog.csdnimg.cn/3b11ec4d6a924f4ebd66ee02dbb6265c.png)

- **协议方案名：**现在大部分更新为https。

- **登陆信息（认证）：**大部分URL都是忽略的。

- **服务器地址：**也就是域名。（因为在网路应用当中，ip地址是纯数字，普通人识别成本高，的所以一般都是将对应的网站涵盖域名，域名 = IP地址，只不过多一步域名解析，（向域名解析服务器）去申请域名所匹配的IP地址）

  > 需要注意的是，我们用IP地址标识公网内的一台主机，但IP地址本身并不适合给用户看。比如说我们可以通过`ping`命令，分别获得`www.baidu.com`和`www.qq.com`这两个域名解析后的IP地址。
  >
  > ![img](https://img-blog.csdnimg.cn/762ebbe1d7334dd1bebcf30771ddb79a.png)
  >
  > 实际我们可以认为域名和IP地址是等价的，在计算机当中使用的时候既可以使用域名，也可以使用IP地址。但URL呈现出来是可以让用户看到的，因此URL当中是以域名的形式表示服务器地址的。

- **服务器端口号：**大部分URL都是省略的。（因为我们所请求的网络服务，对应的端口号都是总所周知的（所有的客户端知道））默认采用的端口号完全对应与我们所使用的服务有关，如http所绑定的端口号为80号端口，https所绑定的端口号为443号端口。

  > `80`表示的是服务器端口号。HTTP协议和套接字编程一样都是位于应用层的，在进行套接字编程时我们需要给服务器绑定对应的IP和端口，而这里的应用层协议也同样需要有明确的端口号。
  >
  > 常见协议对应的端口号：
  >
  > | 协议名称 | 对应端口号 |
  > | -------- | ---------- |
  > | HTTP     | 80         |
  > | HTTPS    | 443        |
  > | SSH      | 22         |

- **带层次的文件路径：**

![img](https://img-blog.csdnimg.cn/d372a20e84194063beaf4063f27aeb86.png)

> 当我们发起网页请求时，本质是获得了这样的一张网页信息，然后浏览器对这张网页信息进行解释，最后就呈现出了对应的网页。
> 
>
> ![img](https://img-blog.csdnimg.cn/5271b3532c0f4368a8e286763b0bc0dc.png)
>
> 我们可以将这种资源称为网页资源，此外我们还会向服务器请求视频、音频、网页、图片等资源。HTTP之所以叫做超文本传输协议，而不叫做文本传输协议，就是因为有很多资源实际并不是普通的文本资源。
>
> 因此在URL当中就有这样一个字段，用于表示要访问的资源所在的路径。此外我们可以看到，这里的路径分隔符是/，而不是\，这也就证明了实际很多服务都是部署在Linux上的。





我们平时的上网（目的）：

1. 我们想获取资源。
2. 我们想上传资源。



- **我们想获取资源：**

> **一张照片，一个视频，一段文字等等 == 资源。**

- **在这个资源没有被我们拿到的时候，在哪里？**

>   **服务器上（Linux上）。**一个服务器上，可能存在很多的资源 **—>** 文件 **—>** 请求资源拿到我们的本地主机上 **—>** 服务进程打开我们要访问的文件，读取该文件，通过网络发送给client。
>
>  <u>然后，打开这个文件，就要先找到这个文件。而Linux中标识一个文件，是通过路劲来标识的。于是便有了带层次的文件路径。</u>
>
> - **融汇贯通的理解：**
>
>   >  IP(唯一的机器) + 端口号(所提供的对应进程) + 路径(客户需要的资源) = 全网具有唯一性
>   >
>   > ```唯一的机器 + 唯一的服务 + 唯一的路径：```
>   >
>   >   定义互联网中唯一的一个资源，url (Uniform Resource Location) 统一资源定位符。
>   >
>   > **所有的资源：**全球范围内，只要找到它的url就能访问该资源。（www：万维网）

---

### 2.1 **urlencode编码 和 urldecode**解码



如果用户想在url中包含url本身用来作为特殊字符的字符，url形式的时候，浏览器会自动给我们进行编码Encode。一般服务端收到之后，需要进行转回特殊字符。

 因为：像 / ? : 等这样的字符，已经被url当做特殊意义理解了，因此这些字符不能随意出现。（某个参数中需要带有这些特殊字符, 就必须先对特殊字符进行转义）



- URL 编码（URL Encoding 编码）是指将特殊字符转换为 URL 可接受的格式的过程，以便在 URL 中传输和显示。这个过程包括将特殊字符转换成特定的编码形式，比如空格会被转换为 %20。
- URL 解码（URL Decoding 解码）则是将经过编码的 URL 字符串还原为原始的字符串的过程。这是为了能够正确读取 URL 中的信息和参数。

Encode 是编码的意思，Decode 是解码的意思



- **转义的规则：**

将需要转码的字符转为16进制，然后从右到左，取4位(不足4位直接处理)，每2位做一位，前面加上%，编码成%XY格式。

![img](https://img-blog.csdnimg.cn/42c5c635db8b4b34a265879765140992.png)

  "+" 被转义成了 "%2B"。汉字也是同样的规制。



- **一个在线编码工具：**

> [UrlEncode编码/UrlDecode解码 - 站长工具 (chinaz.com)](https://tool.chinaz.com/tools/urlencode.aspx)
>
> 选中其中的URL编码/解码模式，在输入C++后点击编码就能得到编码后的结果。
>
> ![img](https://img-blog.csdnimg.cn/53e1c5f225e34b84add522158f20d87f.png)
>
> 再点击解码就能得到原来输入的C++。
>
> ![img](https://img-blog.csdnimg.cn/7f254a52d4e6457ebe2c8232d598dae9.png)

实际当服务器拿到对应的URL后，也需要对编码后的参数进行解码，此时服务器才能拿到你想要传递的参数，解码实际就是编码的逆过程。



---

## 3. http



应用层常见的协议有HTTP和HTTPS，传输层常见的协议有TCP，网络层常见的协议是IP，数据链路层对应就是MAC帧了。其中下三层是由操作系统或者驱动帮我们完成的，它们主要负责的是通信细节。如果应用层不考虑下三层，在应用层自己的心目当中，它就可以认为自己是在和对方的应用层在直接进行数据交互。
![img](https://img-blog.csdnimg.cn/e5d2972261354c5ea53f99e59158aad0.png)

> 下三层负责的是通信细节，而应用层负责的是如何使用传输过来的数据，两台主机在进行通信的时候，应用层的数据能够成功交给对端应用层，因为网络协议栈的下三层已经负责完成了这样的通信细节，而如何使用传输过来的数据就需要我们去定制协议，这里最典型的就是HTTP协议。





HTTP是基于请求和响应的应用层服务，作为客户端，你可以向服务器发起request，服务器收到这个request后，会对这个request做数据分析，得出你想要访问什么资源，然后服务器再构建response，完成这一次HTTP的请求。这种基于request&response这样的工作方式，我们称之为cs或bs模式，其中c表示client，s表示server，b表示browser。

![img](https://img-blog.csdnimg.cn/00d7b9c5c3734194acc47c403a03b8eb.png)



 http协议是应用层的协议，底层采用的叫做TCP。换而言之，就是http在进行正常的request与response之前，已经经历了三次握手的过程。即：建立好了连接，在连接建立好的情况下，双方才才进行正常的通讯。



- 由于HTTP是基于请求和响应的应用层访问，因此我们必须要知道HTTP对应的请求格式和响应格式，```这就是学习HTTP的重点。```

---

### 3.1 **http的宏观结构**

 单纯在报文角度，http可以是基于行的超文本协议。 还会向服务器请求视频、音频、网页、图片等资源。HTTP之所以叫做超文本传输协议，而不叫做文本传输协议，就是因为有很多资源实际并不是普通的文本资源。



---

#### 3.1.1 **http请求报文格式**



HTTP请求由以下四部分组成：

1. 请求行：[请求方法]+[url]+[http版本]
2. 请求报头：请求的属性，这些属性都是以key: value的形式按行陈列的。
3. 空行：遇到空行表示请求报头结束。
4. 请求正文：请求正文允许为空字符串，如果请求正文存在，则在请求报头中会有一个Content-Length属性来标识请求正文的长度。
   

![img](https://img-blog.csdnimg.cn/e03269bf410c4005b1598184a00df279.png)

- **版本：**

请求：client告知server：client用的是哪一个http版本。

响应：server告知client：server用的是哪一个http版本。

如同微信1.0与微信2.0，老版本没有新版本的东西。



- 如何将HTTP请求的报头与有效载荷进行分离？

  > 当应用层收到一个HTTP请求时，它必须想办法将HTTP的报头与有效载荷进行分离。对于HTTP请求来讲，这里的请求行和请求报头就是HTTP的报头信息，而这里的请求正文实际就是HTTP的有效载荷。
  >
  > 我们可以根据HTTP请求当中的空行来进行分离，当服务器收到一个HTTP请求后，就可以按行进行读取，如果读取到空行则说明已经将报头读取完毕，实际HTTP请求当中的空行就是用来分离报头和有效载荷的。
  >
  > 如果将HTTP请求想象成一个大的线性结构，此时每行的内容都是用\n隔开的，因此在读取过程中，如果连续读取到了两个\n，就说明已经将报头读取完毕了，后面剩下的就是有效载荷了。

---

#### 3.1.2 **http响应报文格式**



HTTP响应由以下四部分组成：

1. 状态行：[http版本]+[状态码]+[状态码描述]
2. 响应报头：响应的属性，这些属性都是以key: value的形式按行陈列的。
3. 空行：遇到空行表示响应报头结束。
4. 响应正文：响应正文允许为空字符串，如果响应正文存在，则响应报头中会有一个Content-Length属性来标识响应正文的长度。比如服务器返回了一个html页面，那么这个html页面的内容就是在响应正文当中的。
   

![img](https://img-blog.csdnimg.cn/e7da3d9e45314365802b499d6be5aba6.png)



- **建立一个共识：**

> 考虑协议：需要从该协议是如何封装、解包、向上交付的。





- **http是如何区分报头和有效载荷的？**

>
> 对于HTTP响应来讲，这里的状态行和响应报头就是HTTP的报头信息，而这里的响应正文实际就是HTTP的有效载荷。与HTTP请求相同，当应用层收到一个HTTP响应时，也是根据HTTP响应当中的空行来分离报头和有效载荷的。当客户端收到一个HTTP响应后，就可以按行进行读取，如果读取到空行则说明报头已经读取完毕。
>



- **如何得知正文的大小？**

>  报头当中，就涵盖有一种属性```Content-Length：```正文长度，所以只要读完报头，就能够知道正文有多大了。

---







##### 获取浏览器的HTTP请求

在网络协议栈中，应用层的下一层叫做传输层，而HTTP协议底层通常使用的传输层协议是TCP协议，因此我们可以用套接字编写一个TCP服务器，然后启动浏览器访问我们的这个服务器。

由于我们的服务器是直接用TCP套接字读取浏览器发来的HTTP请求，此时在服务端没有应用层对这个HTTP请求进行过任何解析，因此我们可以直接将浏览器发来的HTTP请求进行打印输出，此时就能看到HTTP请求的基本构成。

![img](https://img-blog.csdnimg.cn/a79fd0e38cb14039a70e38274d9c8c95.png)

**一个简易的TCP模型服务器**

- log.hpp

  ```c++
  #pragma once
  #include <iostream>
  #include <ctime>
  #include <cstdarg>
  
  // 日志是有日志级别的
  #define DEBUG 0
  #define NORMAL 1  // 正常
  #define WARNING 2 // 警告 -- 没出错
  #define ERROR 3   // 错误 -- 不影响后续执行（一个功能因为条件等，没有执行）
  #define FATAL 4   // 致命 -- 代码无法继续向后执行
  
  const char *gLevelMap[] =
      {
          "DEBUG",
          "NORMAL",
          "WARNING",
          "ERROR",
          "FATAL",
  };
  
  #define LOGFILE "./threafpool.log"
  
  // 完整的日志功能，至少：日志等级 时间 日志内容 支持用户自定义
  void logMessage(int level, const char *format, ...)
  {
  #ifndef DEBUG_SHOW
      if (level == DEBUG)
          return;
  #endif
      char stdBuffer[1024];             // 标准部分
      time_t timestamp = time(nullptr); // 获得时间截
  
      // struct tm* localtime = localtime(&timestamp); //- 详细时间输出操作
      // localtime->tm_year; ...
      //  ...
  
      snprintf(stdBuffer, sizeof(stdBuffer), "[%s %ld]", gLevelMap[level], timestamp);
  
      char logBuffer[1024]; // 自定义部分
  
      va_list args;
      va_start(args, format);
      // 这个时候就有一个可变参数列表的起始地址
  
      // 可以直接向屏幕中直接打印
      // vprintf(format, args);
      // 向缓冲区logBuffer中打印
      vsnprintf(logBuffer, sizeof logBuffer, format, args);
      va_end(args);
  
      // 向屏幕打印
      // printf("%s%s\n", stdBuffer, logBuffer);
  
      // 向文件打印
      FILE *fp = fopen(LOGFILE, "a+");
      fprintf(fp, "%s%s\n", stdBuffer, logBuffer);
      fclose(fp);
  }
  
  ```

  

- Sock.hpp

 TCP套接字操作的封装。 

```c++
#pragma once

#include <iostream>
#include <string>
#include <cstring>
#include <unistd.h>
#include <memory>
#include "log.hpp"

// 网络四件套
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

class Sock
{
    const static int gbacklog = 20; // 一般不能太大也不能太小
public:
    Sock() {}

    int Socket()
    {
        // 1. 调用socket, 创建文件描述符
        _listensock = socket(AF_INET, SOCK_STREAM, 0);
        if (_listensock < 0)
        {
            logMessage(FATAL, "create socker error, %d:%s", errno, strerror(errno));
            exit(2);
        }

        logMessage(NORMAL, "create socket success, _listensock: %d", _listensock);
        return _listensock;
    }

    void Bind(int sock, uint16_t port, std::string ip = "0.0.0.0")
    {
        // 2. 调用bind, 将当前的文件描述符和ip/port绑定在一起
        struct sockaddr_in local;
        local.sin_family = AF_INET;
        local.sin_port = htons(port);
        // local.sin_addr.s_addr = ip.empty() ? INADDR_ANY : inet_addr(ip.c_str());
        inet_pton(AF_INET, ip.c_str(), &local.sin_addr);

        if (bind(sock, (struct sockaddr *)&local, sizeof(local)) < 0)
        {
            logMessage(FATAL, "bind error, %d:%s", errno, strerror(errno));
            exit(3);
        }
        logMessage(NORMAL, "bind success ");
    }

    void Listen(int sock)
    {
        // 3. 调用listen, 声明当前这个文件描述符作为一个服务器的文件描述符
        if (listen(sock, gbacklog) < 0)
        {
            logMessage(FATAL, "listen error, %d:%s", errno, strerror(errno));
            exit(4);
        }
        logMessage(NORMAL, "create server listen success");
    }

    // 一般而言
    // const std::string &：输入型参数
    // std::string *：输出型参数
    // std::string &：输入输出型参数
    int Accept(int listensock, std::string *ip, uint16_t *port) // 这样既拿出来了新获得的套接字，又将客户端的ip和port拿到了
    {
        //4. 调用accecpt, 并阻塞, 等待客户端连接过来  
        struct sockaddr_in src;
        socklen_t len = sizeof(src);
        int serversock = accept(listensock, (struct sockaddr *)&src, &len);
        if (serversock < 0)
        {
            logMessage(ERROR, "accept error, %d:%s", errno, strerror(errno));
            return -1;
        }

        if(port) *port = ntohs(src.sin_port);
        if(ip) *ip = inet_ntoa(src.sin_addr);
        return serversock;
    }


    //客户端需要调用connect()连接服务器
     bool Connect(int sock, const std::string &server_ip, const uint16_t &server_port)
     {
        struct sockaddr_in server;
        server.sin_family = AF_INET;
        server.sin_port = htons(server_port);
        inet_pton(AF_INET,server_ip.c_str(),&server.sin_addr);
        
        //connect和bind的参数形式一致, 区别在于bind的参数是自己的地址, 而connect的参数是对方的地址; 
        if(connect(sock,(struct sockaddr*)&server,sizeof(server)) == 0) return true;
        else return false;
        logMessage(NORMAL,"connect succed\n");
     }

     ~Sock() {}

private:
    int _port;
    std::string _ip;
    int _listensock;
};
```

---

- httpserver.hpp

```c++
#pragma once

#include <iostream>
#include "Sock.hpp"

class HttpServer
{
public:
    using Func_t = std::function<void(int)>;

public:
    HttpServer(const uint16_t &port, Func_t func)
        : _port(port), _func(func)
    {
        _listensock = _sock.Socket();
        _sock.Bind(_listensock, _port);
        _sock.Listen(_listensock);
    }

    void Start()
    {
        while(true)
        {
            std::string clienip;
            uint16_t clientport = 0;
            int sockfd = _sock.Accept(_listensock,&clienip,&clientport);
            if(sockfd < 0)
                continue;
            if(fork() == 0) 
            {
                close(_listensock);
                _func(sockfd);
                close(sockfd);
                exit(0);
            }
            close(sockfd);
        }
    }

    ~HttpServer()
    {
        if (_listensock >= 0)
            close(_listensock);
    }

private:
    int _listensock;
    uint16_t _port;
    Sock _sock;
    Func_t _func;
};
```

---

- httpserver

```c++
#include <iostream>
#include "httpServer.hpp"
#include <memory>
#include "Usage.hpp"

void HandlerHttpRequst(int sock)
{
    char buffer[1024];
    ssize_t s = recv(sock, buffer, sizeof(buffer) - 1, 0);
    if (s > 0)
    {
        buffer[s] = 0;
        std::cout << buffer << "------------------------\n"
                  << std::endl;
    }
}
int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
    }
    uint16_t port = atoi(argv[1]);
    std::unique_ptr<HttpServer> httpserver(new HttpServer(port, HandlerHttpRequst));
    httpserver->Start();
    return 0;
}
```

运行服务器程序后，然后用浏览器进行访问，此时我们的服务器就会收到浏览器发来的HTTP请求，并将收到的HTTP请求进行打印输出。将服务器在自己的Linux下启动，然后结合ip:port通过浏览器访问。

![image-20240229122900753](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229122900753.png)

![image-20240229123141890](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229123141890.png)

![img](https://img-blog.csdnimg.cn/9f764670a1dc486ca8ef4e2db83418cf.png)



说明一下：

- 浏览器向我们的服务器发起HTTP请求后，因为我们的服务器没有对进行响应，此时浏览器就会认为服务器没有收到，然后再不断发起新的HTTP请求，因此虽然我们只用浏览器访问了一次，但会受到多次HTTP请求。

- 由于浏览器发起请求时默认用的就是HTTP协议，因此我们在浏览器的url框当中输入网址时可以不用指明HTTP协议。

- url当中的/不能称之为我们云服务器上根目录，这个/表示的是web根目录，这个web根目录可以是你的机器上的任何一个目录，这个是可以自己指定的，不一定就是Linux的根目录。

  



- 我们也可以浏览器进行对应的资源申请。

![img](https://img-blog.csdnimg.cn/1cd443f965bd4afa8ed2c6d80215890b.png)



如果我们什么都没有写，那么就会如上是根目录。其并不是我们linux的根目录路，而是叫做web根目录，其是我们可以自定义路径的。

实际我们在进行网络请求的时候，如果不指明请求资源的路径，此时默认你想访问的就是目标网站的首页，也就是web根目录下的index.html文件。

---

- **如何自定义呢？**

  >   比如说：我们在当前路劲之下创建一个wwwroot的文件夹。如此之后，我们就可以将所有的网页资源放在该路径之下。
  >
  > ​    此处我们简易的提取出，访问方想访问的资源。

---



##### 构建HTTP响应给浏览器



- Util.hpp

-  是一个工具类，可以使用其进行各种字符串的处理。

```c++
#pragma once
 
#include <string>
#include <vector>
 
class Util
{
public:
    static void cutString(std::string s, const std::string &sep, std::vector<std::string> *out)
    {
        std::size_t start = 0;
        while(start < s.size())
        {
            auto pos = s.find(sep, start);
            if(pos == std::string::npos) break;
            out->push_back(s.substr(start, pos - start));
            start += pos - start;
            start += sep.size();
        }
        if(start < s.size()) out->push_back(s.substr(start));
    }
};
```



- httpserver.cc

```c++
#include <iostream>
#include "httpServer.hpp"
#include <memory>
#include "Usage.hpp"
#include "util.hpp"

void HandlerHttpRequst(int sock)
{
    char buffer[1024];
    ssize_t s = recv(sock, buffer, sizeof(buffer) - 1, 0);
    if (s > 0)
    {
        buffer[s] = 0;
        std::cout << buffer << "------------------------\n"
                  << std::endl;
    }

    // 2.试着构建一个http的响应
    std::string HttpResponse = "HTTP/1.1 200 OK\r\n";
    HttpResponse += "\r\n";
    HttpResponse += "<html><h3> hello world </h1></html>";
    // 此处写的是一个静态的，这个内容其实不应该硬编入代码里，而是应该以网页的形式存放在wwwroot的目录下
    // 然后让对应的用户去请求这个网页
    // 如果当前用户请求的是根目录，默认代表的是请求我们所对应的web服务器的默认首页

    // 提出每一行
    std::vector<std::string> vline;
    Util::cutString(buffer, "\r\n", &vline);
    printf("----------- 每一行 -----------\n");
    for (auto &iter : vline)
    {
        std::cout << iter << '\n'
                  << std::endl;
    }
    printf("----------- 每一行 -----------\n");

    // 提出第一行的' '隔开的数据
    std::vector<std::string> vblock;
    Util::cutString(vline[0], " ", &vblock);
    printf("----------- 第一行 -----------\n");
    for (auto &iter : vblock)
    {
        std::cout << iter << '\n'
                  << std::endl;
    }
    printf("----------- 第一行 -----------\n");

    // 提出第一行中所想的访问路径
    std::string file = vblock[1];
    printf("----------- 第一行 -----------\n");
    std::cout << file << '\n'
              << std::endl;
    printf("----------- 第一行 -----------\n");

    send(sock, HttpResponse.c_str(), HttpResponse.size(), 0);
}
int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
    }
    uint16_t port = atoi(argv[1]);
    std::unique_ptr<HttpServer> httpserver(new HttpServer(port, HandlerHttpRequst));
    httpserver->Start();
    return 0;
}
```

![image-20240229130624389](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229130624389.png)

![image-20240229130614007](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229130614007.png)

---



- 这里我们可以给浏览器返回一个固定的HTTP响应。我们就将当前服务程序所在的路径作为我们的web根目录，我们可以在该目录下创建一个html文件，然后编写一个简单的html作为当前服务器的首页。

- 当浏览器向服务器发起HTTP请求时，不管浏览器发来的是什么请求，我们都将这个网页响应给浏览器，此时这个html文件的内容就应该放在响应正文当中，我们只需读取该文件当中的内容，然后将其作为响应正文即可。

**进行一个简易的网页路径访问：**

在我们定义的web根目录```"./wwwroot"```里定义所可以进行请求的资源：

![image-20240229130744116](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229130744116.png)

![image-20240229143812638](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229143812638.png)





- **HttpServer.cc**

```c++
#include <iostream>
#include <vector>
#include <string>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <fstream>
#include "httpServer.hpp"
#include <memory>
#include "Usage.hpp"
#include "util.hpp"

// 一般http都要有自己的web根目录
#define ROOT "./wwwroot"
// 如果客户端只请求了一个/,我们返回默认首页
#define HOMEPAGE "index.heml"

void HandlerHttpRequst(int sock)
{
    char buffer[1024];
    ssize_t s = recv(sock, buffer, sizeof(buffer) - 1, 0);
    if (s > 0)
    {
        buffer[s] = 0;
        std::cout << buffer << "------------------------\n"
                  << std::endl;
    }

    // 提出每一行
    std::vector<std::string> vline;
    Util::cutString(buffer, "\r\n", &vline);

    // 提出第一行的' '隔开的数据
    std::vector<std::string> vblock;
    Util::cutString(vline[0], " ", &vblock);

    // 提出第一行中所想的访问路径
    std::string file = vblock[1];
    std::string target = ROOT;
    if (file == "/")
        file = "/index.html";
    target += file;
    std::cout << "访问的路径" << target << std::endl;

    std::string content;
    std::ifstream in(target); // 打开该文件
    if (in.is_open())
    {
        std::string line;
        while (std::getline(in, line))
        {
            content += line;
        }
        in.close();
    }

    std::string HttpResponse;
    if (content.empty())
        HttpResponse = "HTTP/1.1 404 NotFound\r\n";
    else
        HttpResponse = "HTTP/1.1 200 OK\r\n";
    HttpResponse += "\r\n";
    HttpResponse += content;
    send(sock, HttpResponse.c_str(), HttpResponse.size(), 0);

    // std::
    // send(sock, HttpResponse.c_str(), HttpResponse.size(), 0);
}
int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
    }
    uint16_t port = atoi(argv[1]);
    std::unique_ptr<HttpServer> httpserver(new HttpServer(port, HandlerHttpRequst));
    httpserver->Start();
    return 0;
}
```



因此当浏览器访问我们的服务器时，服务器会将这个index.html文件响应给浏览器，而该html文件被浏览器解释后就会显示出相应的内容。

![image-20240229132654041](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229132654041.png)



![image-20240229132815689](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240229132815689.png)



**融会贯通的理解：**Http本质上就是超文本分析。



- 此外，我们也可以通过`telnet`命令来访问我们的服务器，此时也是能够得到这个HTTP响应的。


---



# 4. **HTTP的方法**

我们平时的上网行为其实就两种：

1. 从服务端拿下来资源数据（打开微信、淘宝、拼多多，会得到对应的网站页面）。
2. 把客户端的数据提交到服务器（登录、注册、搜索）。

在Http这里，主要针对的就是资源。资源：通常是一些用户可以直接感知，并理解的东西，比如说我们拿了一张图片、一个视频、一个音频。



- **需要注意：**

  >  ```在TCP套接字中就有所提到，我们在进行正常上网的时候，其实并不是我们在上网，而是我们通过我们本地的一些APP / 浏览器来向服务端发起请求。也就是叫做服务器，通常其以守护进程的方式一直在运行。```
  >
  > 
  >
  > ```而平时我们所上网使用的APP / 浏览器，最终在手机 / 电脑上都叫做一个进程。所以：服务器 -> 服务器进程， 客户端 -> 客户端进程，它们之间进程拿下来数据和将数据提交上去，这个行为的本质就是进程间通讯。只不过咋子Http这里，其通讯需要更具体的通讯方案。```



- **对应的方法：**

1. 从服务端拿下来资源数据 -> 一般都叫做GET方法，也就是Http发起请求时所一般填写的请求方法中所填写的方法。
2. 把客户端的数据提交到服务器 -> 一般都叫做POST方法 / GET方法。



- 融会贯通的理解：

        在一般的Http应用场景中，80% ~ 90% 的请求方法就是两个，其他方法要么被服务端禁用，要么服务端不支持。因为web服务器推送的资源是直接推送给客户的，而客户也有好的客户和非法的客户。所以直接应对客户的服务器，其要以最小的接口，最小的成本能给到用户提供最基本的服务之外，还要尽量减少其自身无用服务的暴露，进而减少服务器其自己出现漏洞的可能性。


- **手动尝试GET方法：**

  > ![img](https://img-blog.csdnimg.cn/b108cd81ca7d4bc7bd91b28f918995e0.png)
  >
  > 其下面所显示的一堆![img](https://img-blog.csdnimg.cn/6fe3c5bb4deb4c5eb8df38a944221690.png)就是传说中的html，就是我们在百度中可以查看到的：
  >
  > ![img](https://img-blog.csdnimg.cn/f04039e29b9f482b90ba0b0fe11d0b16.png)
  >
  > 或者可以，以更多工具 -> 开发者工具的形式进行打开：



 之所以我们所看到，我们抓下来的首页是乱的，而我们所查的页面的网页，格式比起来好很多很多。这是因为浏览器给我们做了格式化处理，其实一般的网页真正给我们看的就是我们抓下来的那个样子。因为其是没有空行、回车等的格式控制的，因为对于此些格式控制，它们也是字符，所以一般网页在进行发布的时候都是要对其进行压缩的，说白了就是一个网页里不需要的符号就直接去掉了，用以减少网络传输的成本。
![img](https://img-blog.csdnimg.cn/c292d9cec6394a7081b179d1dc9f9b67.png)

 

- 如果我们想利用我们自己的代码测试一下GET方法拿网页这个过程，我们就必须使用***\*表单\****来进行测试。

### 4.1 表单

搜集用户的数据，并把用户数据推送给服务器。我们对应的拿数据 / 提交数据对应的方法，其中这两个方法要提交数据给服务器，就必须依托于网页的表单，而表单的作用就是给用户提供对应的输入框和提交按钮。让用户将信息填充好了后，并且把数据推送给服务器，具体的推送其实就是，对应的表单当中的数据，会被转换成http request的一部分。

![img](https://img-blog.csdnimg.cn/f3d5b79ac0f24670a36950aef904f866.png)

上述图片就叫做input输入框。

![img](https://img-blog.csdnimg.cn/d8b7d3e1ebb141e7b036ea38d5a452c9.png)

---





**index.html**

####  GET方法

```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTTP学习测试</title>
</head>
 
<body>
    <h3>这是一次学习</h3>
    <form name="input" action="/a/b/haha.html" method="get">
        Username: <input type="text" name="user">
        Password: <input type="password" name="pwd">
        <input type="submit" value="登陆">
    </form>
</body>
 
</html>
```

![image-20240301131531585](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240301131531585.png)

**我们进行对应的登录可以发现：**

![img](https://img-blog.csdnimg.cn/3a94b43f28fa4584bc8ef8e9b8c0a90e.gif)

![img](https://img-blog.csdnimg.cn/92625dcc8495431abc0cef6f66cc423a.png)



**跳转地址分析：** 

![img](https://img-blog.csdnimg.cn/1a375432054c4455b1dba6f12e582ae3.png)

如：在百度上进行搜索。

![img](https://img-blog.csdnimg.cn/13b51f2cbca1474c8fbdaed2d4c999ef.png)

而，当我们一点击对应的登陆的时候，我们最终的参数是直接就会回显到url当中的输入框的

所以GET方法最重大的特性就是：它会将我们所对应的参数直接回显到我们所对应的url向服务端传参。



---



**index.html**

#### POST方法

```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTTP学习测试</title>
</head>
 
<body>
    <h3>这是一次学习</h3>
    <form name="input" action="/a/b/haha.html" method="post">
        Username: <input type="text" name="user">
        Password: <input type="password" name="pwd">
        <input type="submit" value="登陆">
    </form>
</body>
 
</html>
```

![img](https://img-blog.csdnimg.cn/37fd3807636e476c803c4404a688fffe.gif)

![img](https://img-blog.csdnimg.cn/fff46b6678224807add00bd05833585b.png)

我们可以发现它将参数放到了正文的部分：

![img](https://img-blog.csdnimg.cn/e752fee809ef48f6b2e4a0c8a8977955.png)

POST通过http的正文方式提交参数。



---

#### **POST与GET的区别：**

 将数据从客户端提交向服务器时，它们两个提交参数是不一样的。

- GET方法通过url传参，回显输入的私密信息，不够私密。
- POST方法通过正文提交参数，不会回显，一般私密性是有保证的。



> Note：
>         私密不是安全，加密和解密才是安全的。

不成文的规定：当上传的东西过大，使用POST方法传，不建议使用GET方法。主要也是因为：```GET方法```使用url进行传参，其是按行为单位的，参数里如果包含换行也就不好处理，并且解析起来不方便，还要我们自己解析。而如果使用```POST方法```，其报头里是有```Content-Length```字段标识正文长度的，是很方便我们进行整体内容的读取。





- **HTTP的状态码**

| 类别 | 类型             | 描述                       |
| ---- | ---------------- | -------------------------- |
| 1XX  | 信息性状态码     | 接收的请求正在处理         |
| 2XX  | 成功状态码       | 请求正常处理完毕           |
| 3XX  | 重定向状态码     | 需要进行附加操作以完成请求 |
| 4XX  | 客户端错误状态码 | 服务器无法处理请求         |
| 5XX  | 服务器错误状态码 | 服务器处理请求出错         |

最常见的状态码, 比如 200(OK), 404(Not Found), 403(Forbidden), 302(Redirect, 重定向 ), 504(Bad Gateway)

---

### 4.2 **重定向**



- **重定向：**

> 当我们进行某些网站请求时 / 某些网页请求时，因为功能的要求，要求我们去请求其他网页 / 网站。

- **永久重定向 (301) VS 临时重定向 (302 / 307)：**



- **临时重定向：**

>   突然有一天，该店铺说：由于装修原因，于是将店铺**临时**移动到一个附近的新地址。于是当你看到后，会跑去新的地址，但是下回你再来，还是会来到老店铺看一眼，是否开店。

- **永久重定向：**

>   突然有一天，该店铺说：由于老旧原因，于是将店铺**永久**移动到一个附近的新地址。于是当你看到后，会跑去新的地址，并且下回你再也不会到老店铺，而是直接到新店铺。



**总结：**

- 临时重定向：不影响用户后续的请求策略。
- 永久重定向：影响用户的后续的请求策略。



- **临时重定向：**

> 当我们对对应的网站地址进行请求之后，会返回302，并告诉我们新的地址，然后浏览器会根据对应的新地址，自行执行跳转。 
>
> ![img](https://img-blog.csdnimg.cn/698016020885429aa73076a123315a7c.png)



- **临时重定向部分的编写：** 

>  将不存在的路径，重定向到```http://www.qq.com```
>
> ```c++
>     std::string HttpResponse;
>     if(content.empty())
>     {
>         HttpResponse = "HTTP/1.1 302 Found\r\n";
>         HttpResponse += "Location: https://www.qq.com/\r\n"; // 临时重定向到qq
>     }
>     else HttpResponse = "HTTP/1.1 200 OK\r\n";
>     HttpResponse += "\r\n";
>     HttpResponse += content;
>     send(sockfd, HttpResponse.c_str(), HttpResponse.size(), 0);
> ```
>
> 

![](C:\Users\。\Desktop\---\笔记Gif作图\PixPin_2024-03-01_14-12-44.gif)





- 如果我们想实现一个在站内的跳转，也就是说当访问的资源不存在的时候，就返回一个404的页面：

```c++
 std::string HttpResponse;
    if (content.empty())
    {
         HttpResponse = "HTTP/1.1 302 Found\r\n";
         HttpResponse += "Location: http://123.207.8.23:8081/a/404.html\r\n";
    }

    else
    {
        HttpResponse = "HTTP/1.1 200 OK\r\n";
    }
    HttpResponse += "\r\n";
    HttpResponse += content;
    send(sock, HttpResponse.c_str(), HttpResponse.size(), 0);
```

![image-20240301144643175](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240301144643175.png)

**404.html**

```c++
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>不存在</title>
</head>
<body>
    <h2>你访问的页面不存在</h2>
</body>
</html>
```

![img](https://img-blog.csdnimg.cn/61c934daeeda42b2a8bebd429709e205.gif)

>  **301：**相对于302多做一部分的工作，也就是更新一下本地的缓存 / 书签。以此达到后续的访问。

---



# 5. **HTTP常见Header**

- **Content-Length:** 报文的长度。
- **Content-Type:** 正文的数据类型(text / html等)。

​		  Content-Type对照表的查询：[Content-Type对照表 - MKLab在线工具](https://www.mklab.cn/docs/contenttype)

 但是我们可以发现，我们写与不写，好像根本没有什么区别。这是因为现在的浏览器都非常的聪明，当我们在请求的时候，即便不写清**Content-Type**，浏览器也可以收到正文之后，也可以根据正文的内容，对文件内容做自动识别。注意：这个识别并不是所有的浏览器都可以识别到的。



 如果我们将给予的网络响应，却告诉浏览器是存文本的：

```c++
    std::string HttpResponse;
    if(content.empty())
    {
        HttpResponse = "HTTP/1.1 302 Found\r\n";
        HttpResponse += "Location: http://123.207.8.23:8081/a/404.html\r\n";
    }
    else 
    {
        HttpResponse = "HTTP/1.1 200 OK\r\n";
        HttpResponse += ("content: text/plain\r\n");
        HttpResponse += ("content-Length: " + std::to_string(content.size()) + "\r\n");
    }
    HttpResponse += "\r\n";
    HttpResponse += content;
    send(sockfd, HttpResponse.c_str(), HttpResponse.size(), 0);
```

- **Host:** 客户端告知服务器，所请求的资源是在哪个主机的哪个端口上。

- **User-Agent:** 声明用户的操作系统和浏览器版本信息。

![img](https://img-blog.csdnimg.cn/970a06acd4fc49d98ed2221290852552.png)

 可以发现，我们用电脑的浏览器所搜索下载出现的推荐，与手机浏览器所搜索下载出现的推荐是不同的。就是因为**User-Agent**的存在，对应的目标网站会根据我们说发送的请求识别**User-Agent**知晓我们所使用的操作系统信息。

- **referer:** 当前页面是从哪个页面跳转过来的。

- **location:** 搭配3xx状态码使用，告诉客户端接下来要去哪里访问。
- **Cookie:** 用于在客户端存储少量信息. 通常用于实现会话（session）的功能。

---



### 5.1 **会话管理**



- 简单快速

>   应用层http协议请求的是在互联网当中的各种资源，底层采用TCP + 套接字帮我们去请求，请求的时候会告诉服务端需要什么资源（通过http的request中的状态行中的url）。



- 无连接

> http协议这一层，http是不维护链接的，是TCP协议来维护的。



- 无状态

> 说白了就是：http协议其并不会记录一个用户，历史上，也就是上一次的请求，不会对用户的行为做任何的记录，不会认为当前用户历史上有没有登录过，有没有直接访问过该网页。

> **一个理解：**
>
> ​    我们如使用淘宝、哔哩哔哩等网页，我们需要登录，而且登录与主页并不是一个页面。由于http协议的无状态，于是压根就没有登录的记录。

- **这个无状态的情况就会是个很难堪的局面？**

> 然而平时的使用告诉我们，并不是这样的。（以网页哔哩哔哩为例）真实情况是我们将网页关掉再重新进入，登录还是存在，甚至将浏览器关掉再开也是登录情况。

- **这么如何理解？**

>    http协议确实不记录状态 ，但是并不代表网站服务，不会为我们提供这样的服务。因为我们的web服务器，我们所写的底层套接字，都是我们所写的，我们并没有对用户的状态进行管理，因为这个事本身就不能交给协议去做。其应该是上层的业务逻辑去维护的，所以http协议不管心。它就关系文件大小、文件类别等，负责网络特性。
>



---

### 5.2. **Cookie**



**此状态是怎么记录的？**?

 为了支持网站进行常规用户的会话管理，报头属性里会有两种属性：

![img](https://img-blog.csdnimg.cn/16cc2d72c275485e85a597edc5e409fc.png)

![img](https://img-blog.csdnimg.cn/29194f08a19945378bfbee3861f2d38e.png)



- **内存级：**浏览器进程关掉就没有记录了。

- **文件级：**浏览器进程关掉还有记录。

  > - 将浏览器关掉后再打开，访问之前登录过的网站，如果需要你重新输入账号和密码，说明你之前登录时浏览器当中保存的cookie信息是内存级别的。
  > - 将浏览器关掉甚至将电脑重启再打开，访问之前登录过的网站，如果不需要你重新输入账户和密码，说明你之前登录时浏览器当中保存的cookie信息是文件级别的。



 并且我们是可以看到cookie文件的：

![img](https://img-blog.csdnimg.cn/14728b3dbcbc42d49ea3264d8e548010.png)



-  现在就有一个问题，浏览器将我们对应的账号和密码都保存起来，如果有一天中了黑客的圈套：**木马！**

> 其直接扫描我们的浏览器的相关的文件，找到我们的cookie文件，盗走文件，然后使用其自身的浏览器打开网站，于是黑客直接使用免密码的方式登录上我们的账号，于是个人信息就严重泄漏了。
>
> 于是在现实生活中，这个方案无法再进行采取，这个方法特别的不安全。



- **一个方案（现在主流）：** 

   cookie文件继续用，但是方案进行调整。

        采用将数据存储在服务器端，而返回一个由算法生成的唯一ID（存储私密信息在服务器端文件的文件名为唯一ID：session id）存储在cookie文件中，之后在session id有效期内，http request时就会自动携带cookie->session id自动登录。

   ![img](https://img-blog.csdnimg.cn/4bae8359f40b4fb38c1abc36229f7305.png)



- **但是这样，好像黑客还是可以通过拿到我们的cookie然后进行无密码登录？**

  > 是的，还是可以通过拿到我们的 cookie 然后进行无密码的登录。但是与上次的区别的就是，黑客最多拿到session id，而个人信息并没有做任何的泄漏。而黑客通过此方法进行无密码的登录，是无法避免的了的，这也就是一堆人QQ被盗的原因（没有十全十美的完美防御，只能尽量避免，因为客户太 "小白" 了）。
  >



- **session id 方案的好处：**
- 曾经因为没有session id：黑客得到cookie，直接得到账户密码，服务端没有任何办法，拿着账户密码正确，阻拦也不对。
  
- 曾经因为有session id：黑客得到cookie，也就是得到session id，这次服务端站的主权。主要盗取都是外国的，所以ip就会异常，于是对应公司的账户安全团队就会识别到：异地访问，于是直接将session id立即设置为失效，然后向绑定手机发送短信、需要重新登录。

---

#### 5.2.1 **实验证明**

```c++
    std::string HttpResponse;
    if(content.empty())
    {
        HttpResponse = "HTTP/1.1 302 Found\r\n";
        HttpResponse += "Location: http://121.36.24.39:8080/a/b/c/404.html\r\n";
    }
    else 
    {
        HttpResponse = "HTTP/1.1 200 OK\r\n";
        HttpResponse += ("content-Type: text/plain\r\n");
        HttpResponse += ("content-Length: " + std::to_string(content.size()) + "\r\n");
        HttpResponse += "Set-Cookie: 这是一个cookie\r\n";
    }
    HttpResponse += "\r\n";
    HttpResponse += content;
    send(sockfd, HttpResponse.c_str(), HttpResponse.size(), 0);
```

![img](https://img-blog.csdnimg.cn/a1d3c1e73d154524aa3ac706ce81c42a.png)

点击登录后

![img](https://img-blog.csdnimg.cn/b609b4d21e7f462e8339293091a9b77b.png)



- **客户端是如何将信息提交向服务器？服务器又是如何将cookie写给客户端的？**

![img](https://img-blog.csdnimg.cn/16cc2d72c275485e85a597edc5e409fc.png)



### 5.3 **Connection选项**

- **Connection: close**，代表在连接的时候，采用短链接。
- **Connection: keep-alive**，代表在连接的时候，采用长链接。



一个完整的网页，是由非常多的资源构成的。

- **短链接：**发动十次请求，TCP层面浏览器就要发起十次连接 —— 成本高。
- **长链接：**发动十次请求，一次连接搞定 —— 成本低。

也正因为报头与正文的格式规范化，我们一次读取多个请求也是没有任何问题的。



HTTP/1.0是通过request&response的方式来进行请求和响应的，HTTP/1.0常见的工作方式就是客户端和服务器先建立链接，然后客户端发起请求给服务器，服务器再对该请求进行响应，然后立马端口连接。

但如果一个连接建立后客户端和服务器只进行一次交互，就将连接关闭，就太浪费资源了，因此现在主流的HTTP/1.1是支持长连接的。所谓的长连接就是建立连接后，客户端可以不断的向服务器一次写入多个HTTP请求，而服务器在上层依次读取这些请求就行了，此时一条连接就可以传送大量的请求和响应，这就是长连接。

如果HTTP请求或响应报头当中的Connect字段对应的值是Keep-Alive，就代表支持长连接。

---



# 6. **工具推荐**

 

- **postman：**可以让我们的客户端，手动的向目标服务器构建Http请求。
- **Fiddler：**是一个进行抓包的工具，我们所有出去的网络请求，最终Fiddler都可以帮我们抓到，主要抓Http。

---

### **Fiddler**

  Fiddler 4安装与配置博客推荐：[Fiddler4安装与配置_偷懒的肥猫的CSDN博客](https://blog.csdn.net/weixin_45135767/article/details/108401189)

  Fiddler将我们本主机的http请求全部抓取到。

---

#### **原理**

 原本的浏览器访问服务器。

![img](https://img-blog.csdnimg.cn/d30efd9c29934460a23348d25f4d04a9.png)



当有**fidder**之后，实际上请求是首先发送给**fiddler**（相当于截止），**fiddler**拿到了浏览器的请求，然后**fiddler**帮浏览器进行请求，响应结果传递原理同理。

![img](https://img-blog.csdnimg.cn/3573677699234fe9b199f91db26d28eb.png)



我们通过使用浏览器对我们所写的服务端进行请求，然后使用fiddler进行捕捉会发现：

![img](https://img-blog.csdnimg.cn/7f625b76d1a2415688e3576a78935320.png)



  fiddler捕捉到的请求方法是不一样的，其url带**ip**、带***端口***的，而我们收到的请求是：

![img](https://img-blog.csdnimg.cn/90fd4d428c534962b68384e87c77d1a1.png)



只是GET然后没有带对应的ip地址，就是因为这个http请求，请求的时候已经到对应的主机了（在这个主机上找资源）而fiddler之所以带ip了，是因为fiddler要替我们去请求。
