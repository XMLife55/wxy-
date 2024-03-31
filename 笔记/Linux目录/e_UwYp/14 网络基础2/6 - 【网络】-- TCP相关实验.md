## 1. TCP相关实验

### 理解CLOSE_WAIT状态

当客户端和服务器在进行[TCP通信](https://so.csdn.net/so/search?q=TCP通信&spm=1001.2101.3001.7020)时，如果客户端调用close函数关闭对应的文件描述符，此时客户端底层操作系统就会向服务器发起FIN请求，服务器收到该请求后会对其进行ACK响应。

但如果当服务器收到客户端的FIN请求后，服务器端不调用close函数关闭对应的文件描述符，那么服务器就不会给客户端发送FIN请求，相当于只完成了四次挥手当中的前两次挥手，此时客户端和服务器的连接状态分别会变为FIN_WAIT_2和CLOSE_WAIT。

![img](https://img-blog.csdnimg.cn/0c5321ebb1754a72841374f44361264a.png)



我们可以编写一个简单的TCP套接字来模拟出该现象，实际我们只需要编写服务器端的代码，而采用一些网络工具来充当客户端向我们的服务器发起连接请求。

服务器的初始化需要进行套接字的创建、绑定以及监听，然后主线程就可以通过调用accept函数从底层获取建立好的连接了。获取到连接后主线程创建新线程为该连接提供服务，而新线程只需执行一个死循环逻辑即可。

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <pthread.h>

const int port = 8081;
const int num = 5;

void* Routine(void* arg)
{
	pthread_detach(pthread_self());
	int fd = *(int*)arg;
	delete (int*)arg;
	while (1){
		std::cout << "socket " << fd << " is serving the client" << std::endl;
		sleep(1);
	}

	return nullptr;
}

int main()
{
	//创建监听套接字
	int listen_sock = socket(AF_INET, SOCK_STREAM, 0);
	if (listen_sock < 0){
		std::cerr << "socket error" << std::endl;
		return 1;
	}
	//绑定
	struct sockaddr_in local;
	memset(&local, 0, sizeof(local));
	local.sin_port = htons(port);
	local.sin_family = AF_INET;
	local.sin_addr.s_addr = INADDR_ANY;

	if (bind(listen_sock, (struct sockaddr*)&local, sizeof(local)) < 0){
		std::cerr << "bind error" << std::endl;
		return 2;
	}
	//监听
	if (listen(listen_sock, num) < 0){
		std::cerr << "listen error" << std::endl;
		return 3;
	}
	//启动服务器
	struct sockaddr_in peer;
	memset(&peer, 0, sizeof(peer));
	socklen_t len = sizeof(peer);
	
	for (;;){
		int sock = accept(listen_sock, (struct sockaddr*)&peer, &len);
		if (sock < 0){
			std::cerr << "accept error" << std::endl;
			continue;
		}
		std::cout << "get a new link: " << sock << std::endl;
		int* p = new int(sock);
		
		pthread_t tid;
		pthread_create(&tid, nullptr, Routine, (void*)p);
	}
	return 0;
}

```

代码编写完毕后运行服务器，并用telnet工具连接我们的服务器，此时通过以下监控脚本就可以看到两条状态为ESTABLISHED的连接。

```c++
[cl@VM-0-15-centos DisconnectStatus]$ while :; do sudo netstat -ntp|head -2&&sudo netstat -ntp | grep 8081; sleep 1; echo "##################"; done

```

![img](https://img-blog.csdnimg.cn/5cd23886a34f4eb3a238704cd9b4a35e.png)

这两条连接当中，一条是客户端到服务器的连接，另一条就是服务器到客户端的连接。

现在我们让telnet退出，就相当于客户端向服务器发起了连接断开请求，但此时服务器端并没有调用close函数关闭对应的文件描述符，所以当telnet退出后，客户端维护的连接的状态会变为FIN_WAIT_2，而服务器维护的连接的状态会变为CLOSE_WAIT。

![img](https://img-blog.csdnimg.cn/648b6cd29af44d56bc71570bf6cefeb0.png)

---



### 理解[TIME_WAIT](https://so.csdn.net/so/search?q=TIME_WAIT&spm=1001.2101.3001.7020)状态

当客户端和服务器在进行TCP通信时，客户端调用close函数关闭对应的文件描述符，如果服务器收到后也调用close函数进行了关闭，那么此时双方将正常完成四次挥手。

但主动发起四次挥手的一方在四次挥手后，不会立即进入CLOSED状态，而是进入短暂的TIME_WAIT状态等待若干时间，最终才会进入CLOSED状态

![img](https://img-blog.csdnimg.cn/7e1cb6ac0a9e471aa3b929ee09747536.png)

我们可以继续刚才的实验，由于telnet退出后服务器端没有调用close关闭对应的文件描述符，因此客户端维护的客户端维护连接的状态停留在了FIN_WAIT_2状态，而服务器维护连接的状态停留在了CLOSE_WAIT状态。

要让客户端和服务器继续完成后两次挥手，就需要服务器端调用close函数关闭对应的文件描述符。虽然服务器代码当中没有调用close函数，但因为文件描述符的生命周期是随进程的，当进程退出的时候，该进程所对应的文件描述符都会自动关闭。

因此只需要在telnet退出后让服务器进程退出就行了，此时服务器进程所对应的文件描述符会自动关闭，此时服务器底层TCP就会向客户端发送FIN请求，完成剩下的两次挥手。

四次挥手后客户端维护的连接就会进入到TIME_WAIT状态，而服务器维护的连接则会立马进入到CLOSED状态。

![img](https://img-blog.csdnimg.cn/8cb89c7cac3c4b8e98f8d875ff17bc1b.png)

---



### 解决TIME_WAIT状态引起的bind失败的方法

主动发起四次挥手的一方在四次挥手后，会进入TIME_WAIT状态。如果在有客户端连接服务器的情况下服务器进程退出了，就相当于服务器主动发起了四次挥手，此时服务器维护的连接在四次挥手后就会进入TIME_WAIT状态。![img](https://img-blog.csdnimg.cn/b9cb33acfacf477d829a00fe5997943b.png)

在该连接处于TIME_WAIT期间，如果服务器想要再次重新启动，就会出现绑定失败的问题。

因为在TIME_WAIT期间，这个连接并没有被完全释放，也就意味着服务器绑定的端口号仍然是被占用的，此时服务器想要继续绑定该端口号启动，就只能等待TIME_WAIT结束。



但当服务器崩溃后最重要实际是让服务器立马重新启动，如果想要让服务器崩溃后在TIME_WAIT期间也能立马重新启动，需要让服务器在调用socket函数创建套接字后，继续调用setsockopt函数设置端口复用，这也是编写服务器代码时的推荐做法。



### setsockopt

setsockopt函数可以设置端口复用，该函数的函数原型如下：

```c++
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

```

参数说明：

- sockfd：需要设置的套接字对应的文件描述符。
- level：被设置选项的层次。比如在套接字层设置选项对应就是SOL_SOCKET。
- optname：需要设置的选项。该选项的可取值与设置的level参数有关。
- optval：指向存放选项待设置的新值的指针。
- optlen：待设置的新值的长度。



- **返回值说明：**

  - 设置成功返回0，设置失败返回-1，同时错误码会被设置。

   我们这里要设置的就是监听套接字，将监听套接字在套接字层设置端口复用选项SO_REUSEADDR，该选项设置为非零值表示开启端口复用。

```c++
int opt = 1;
setsockopt(listen_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```



此时当服务器崩溃后我们就可以立马重新启动服务器，而不用等待TIME_WAIT结束。

![img](https://img-blog.csdnimg.cn/5bdc574c8de44bdf84c0c384935b6682.png)



- **连接是由TCP管理的**

  > 从上面的实验中可以看到，即便通信双方对应的进程都退出了，但服务器端依然存在一个处于TIME_WAIT状态的连接，这也更加说明了进程管理和连接管理是两个相对独立的单元。连接是由TCP自行管理的，连接不一定会随进程的退出而关闭。



---

### 理解listen的第二个参数

在编写TCP套接字的服务器代码时，在进行了套接字的创建和绑定之后，需要调用listen函数将创建的套接字设置为监听状态，此后服务器就可以调用accept函数获取建立好的连接了。其中listen函数的第一个参数就是需要设置为监听状态的套接字，而listen的第二个参数我们一般设置为5，那你知道listen函数的第二个参数具体的含义是什么吗？

- 下面通过一个实验来说明listen的第二个参数的具体含义：
  - 先编写TCP套接字的服务器端代码，服务器初始化时依次进行套接字创建、绑定、监听，但服务器初始化后不调用accept函数获取底层建立好的连接。
  - 为了方便验证，这里将listen函数的第二个参数设置为2。

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <pthread.h>

const int port = 8081;
const int num = 2;

int main()
{
	//创建监听套接字
	int listen_sock = socket(AF_INET, SOCK_STREAM, 0);
	if (listen_sock < 0){
		std::cerr << "socket error" << std::endl;
		return 1;
	}
	int opt = 1;
	setsockopt(listen_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
	//绑定
	struct sockaddr_in local;
	memset(&local, 0, sizeof(local));
	local.sin_port = htons(port);
	local.sin_family = AF_INET;
	local.sin_addr.s_addr = INADDR_ANY;
	
	if (bind(listen_sock, (struct sockaddr*)&local, sizeof(local)) < 0){
		std::cerr << "bind error" << std::endl;
		return 2;
	}
	//监听
	if (listen(listen_sock, num) < 0){
		std::cerr << "listen error" << std::endl;
		return 3;
	}
	//启动服务器
	for (;;){
		//不调用accept获取连接
	}
	return 0;
}

```

运行服务器后使用`netstat -nltp`命令，可以看到该服务器当前正处于监听状态。

![img](https://img-blog.csdnimg.cn/c07b876066fa4a01b1a796f9ab5630ac.png)

接下来用Postman向我们的服务器发起一个连接请求。

![img](https://img-blog.csdnimg.cn/280d73ae56894b9a950264c3ccb1013a.png)



**说明一下：** 为了让实验现象更加更加明显，这里最好不要用浏览器进行测试，因为浏览器发起连接请求后如果得不到响应会进行重发。

实验Postman发起连接请求后，此时通过以下命令就可以看到，此时在服务器端新增了一个连接，该连接当前处于ESTABLISHED状态。

```c++
[cl@VM-0-15-centos listenParam]$ sudo netstat -ntp | head -2&&sudo netstat -ntp | grep 8081
```

![img](https://img-blog.csdnimg.cn/a1968e3a1ebf4ff5b6af5bbe2de923d5.png)

继续用Postman发起第二个、第三个连接请求。

![img](https://img-blog.csdnimg.cn/604e84aa236944ca84c4dff0e9c3fc4b.png)

此时在服务器端也会新增对应的第二个、第三个已经建立好的连接，由于服务器没有调用accept函数获取已经建立好的连接，所以现在服务器端已经建立的连接的个数就是3。

![img](https://img-blog.csdnimg.cn/ec215983bb9840baa39c368aa213d186.png)

现在继续使用Postman向服务器发起第四次连接请求。

![img](https://img-blog.csdnimg.cn/af01c4f8cb8941ce870104ff9485efb2.png)

此时在服务器端没有继续新增状态为ESTABLISHED的连接，而是新增了一个状态为SYN_RCVD的连接。

![img](https://img-blog.csdnimg.cn/da989906e93b400891a8bb9b29b9d362.png)

现在服务器端新增了一个状态SYN_RCVD的连接，也就意味着服务器没有对刚才客户端发来的连接请求进行响应。

![img](https://img-blog.csdnimg.cn/d71acdd2d18c4472a87bc3e4f00de477.png)

此时就算再用Postman向服务器发起连接请求，在服务器端也不会再新增任何状态的连接了。

而对于刚才状态为SYN_RCVD的连接，由于服务器长时间不对其进行应答，三次握手失败后该连接会被自动释放。

![img](https://img-blog.csdnimg.cn/6abc29f4e4654607ad69dd98e49bd5ce.png)

此时在Postman这里就会对应看到如下提示，表示向服务器发起的连接请求没有得到响应。

![img](https://img-blog.csdnimg.cn/108647d7d12b49da89df121fb7440d70.png)



- **总结一下上面的实验现象：**

  - 无论有多少客户端向服务器发起连接请求，最终在服务器端最多只有三个连接会建立成功。
  - 当发来第四个连接请求时，服务器只是收到了该客户端发来的SYN请求，但并没有对其进行响应。
  - 当发来更多的连接请求时，服务器会直接拒绝这些连接请求。

  

  ---

#### listen的第二个参数

实际TCP在进行连接管理时会用到两个连接队列：

- 全连接队列（accept队列）。全连接队列用于保存处于ESTABLISHED状态，但没有被上层调用accept取走的连接。
- 半连接队列。半连接队列用于保存处于SYN_SENT和SYN_RCVD状态的连接，也就是还未完成三次握手的连接。

而全连接队列的长度实际会受到listen第二个参数的影响，一般TCP全连接队列的长度就等于listen第二个参数的值加一。

因为我们实验时设置listen第二个参数的值为2，此时在服务器端全连接队列的长度就为3，因此服务器最多只允许有三个处于ESTABLISHED状态的连接。

如果将刚才代码中listen的第二个参数值设置为3，此时服务器端最多就允许存在4个处于ESTABLISHED状态的连接。在服务器端已经有4个ESTABLISHED状态的连接的情况下，再有客户端发来建立连接请求，此时服务器端就会新增状态为SYN_RCVD的连接，该连接实际就是放在半连接队列当中的。

![img](https://img-blog.csdnimg.cn/8603b5da20a446dea9ab5cfdf1223794.png)

此后就算再有客户端发来连接请求，在服务器端也不会新增任何状态的连接。



- **为什么底层要维护连接队列？**

一般当服务器压力较大时连接队列的作用才会体现出来，如果服务器压力本身就不大，那么一旦底层有连接建立成功，上层就会立马将该连接读走并进行处理。

- 服务器端启动时一般会预先创建多个服务线程为客户端提供服务，主线程从底层accept上来连接后就可以将其交给这些服务线程进行处理。
- 如果向服务器发起连接请求的客户端很少，那么连接一旦在底层建立好就被主线程立马accept上来并交给服务线程处理了。
- 但如果向服务器发起连接请求的客户端非常多，当每个服务线程都在为某个连接提供服务时，底层再建立好连接主线程就不能获取上来了，此时底层这些已经建立好的连接就会被放到连接队列当中，只有等某个服务线程空闲时，主线程就会从这个连接队列当中获取建立好的连接。
- 如果没有这个连接队列，那么当服务器端的服务线程都在提供服务时，其他客户端发来的连接请求就会直接被拒绝。
- 但有可能正当这个连接请求被拒绝时，某个服务线程提供服务完毕，此时这个服务线程就无法立马得到一个连接为之提供服务，所以一定有一段时间内这个服务线程是处于闲置状态的，直到再有客户端发来连接请求。
- 而如果设置了连接队列，当某个服务线程提供完服务后，如果连接队列当中有建立好的连接，那么主线程就可以立马从连接队列当中获取一个连接交给该服务线程进行处理，此时就可以保证服务器几乎是满载工作的。



- **为什么连接队列不能太长？**

全连接队列不能太长，系统一般设置为5

虽然维护连接队列能让服务器处于几乎满载工作的状态，但连接队列也不能设置得太长。

- 如果队列太长，也就意味着在队列较尾部的连接需要等待较长时间才能得到服务，此时客户端的请求也就迟迟得不到响应。
- 此外，服务器维护连接也是需要成本的，连接队列设置的越长，系统就要花费越多的成本去维护这个队列。
- 但与其与其维护一个长连接，造成客户端等待过久，并且占用大量暂时用不到的资源，还不如将部分资源节省出来给服务器使用，让服务器更快的为客户端提供服务。

因此虽然需要维护连接队列，但连接队列不能维护的太长。



- **全连接队列的长度**

全连接队列的长度由两个值决定：

- 用户层调用listen时传入的第二个参数backlog。
- 系统变量net.core.somaxconn，默认值为128。

通过以下命令可以查看系统变量net.core.somaxconn的值。

```c++
[cl@VM-0-15-centos listenParam]$ sudo sysctl -a | grep net.core.somaxconn
```

![img](https://img-blog.csdnimg.cn/18fb9c9004b64b2ab579a11967df64ee.png)

全连接队列的长度实际等于listen传入的backlog和系统变量net.core.somaxconn中的较小值加一。

```
ps：半连接队列长度尚未找到非常明确的计算方法.
```

---



### SYN洪水攻击

连接正常建立的过程：

- 当客户端向服务器发起连接建立请求后，服务器会对其进行SYN+ACK响应，并将该连接放到半连接队列（syns queue）当中。
- 当服务器发出的SYN+ACK得到客户端响应后，就会将该连接由半连接队列移到全连接队列（accept queue）当中。
- 此时上层就可以通过调用accept函数，从全连接队列当中获取建立好的连接了。
  

![img](https://img-blog.csdnimg.cn/d9ce1a8a6c5b4a339bbdb2d06a5f6769.png)



**连接建立异常：**

- 但如果客户端在发起连接建立请求后突然死机或掉线，那么服务器发出的SYN+ACK就得不到对应的ACK应答。
- 这种情况下服务器会进行重试（再次发送SYN+ACK给客户端）并等待一段时间，最终服务器会因为收不到ACK应答而将这个连接丢弃，这段时间长度就称为SYN timeout。
- 在SYN timeout时间内，这个连接会一直维护在半连接队列当中。

![img](https://img-blog.csdnimg.cn/5d6a1610d3fc458b87042ff3a3f8efdf.png)

此时服务器虽然需要短暂维护这些异常连接，但这种情况毕竟是少数，不会对服务器造成太大影响。

但如果有一个恶意用户故意大量模拟这种情况：向服务器发送大量的连接建立请求，但在收到服务器发来的SYN+ACK后故意不对其进行ACK应答。

- 此时服务器就需要维护一个非常大的半连接队列，并且这些连接最终都不会建立成功，也就不会被移到全连接队列当中供上层获取，最后会导致半连接队列越来越长。
- 当半连接队列被占满后，新来的连接就会直接被拒绝，哪怕是正常的连接建立请求，此时就会导致正常用户无法访问服务器。
- 这种向服务器发送大量SYN请求，但并不对服务器的SYN+ACK进行ACK响应，最终可能导致服务器无法对外提供服务，这种攻击方式就叫做``SYN洪水攻击（SYN Flood）``。



---

#### 解决SYN洪水攻击

首先这一定是一个综合性的解决方案，TCP作为传输控制协议需要对其进行处理，而上层应用层也要尽量避免遭到SYN洪水攻击。

- 比如应用层可以记录，向服务器发起连接建立请求的主机信息，如果发现某个主机多次向服务器发起SYN请求，但从不对服务器的SYN+ACK进行ACK响应，此时就可以对该主机进行黑名单认证，此后该主机发来的SYN请求一概不进行处理。



- **TCP为了防范SYN洪水攻击，引入了syncookie机制：**
  - 现在核心的问题就是半连接队列被占满了，但不能简单的扩大半连接队列，就算半连接队列再大，恶意用户也能发送更多的SYN请求来占满，并且维护半连接队列当中的连接也是需要成本的。
  - 因此TCP引入了syncookie机制，当服务器收到一个连接建立请求后，会根据这个SYN包计算出一个cookie值，将其作为将要返回的SYN+ACK包的初始序号，然后将这个连接放到一个暂存队列当中。
  - 当服务器收到客户端的ACK响应时，会提取出当中的cookie值进行对比，对比成功则说明是一个正常连接，此时该连接就会从暂存队列当中移到全连接队列供上层读取。

![img](https://img-blog.csdnimg.cn/90a1183fbf024e5ba4594f4ce0b202a9.png)

引入了syncookie机制的好处：

- 引入syncookie机制后，这些异常连接就不会堆积在半连接队列队列当中了，也就不会出现半连接队列被占满的情况了。
- 对于正常的连接，一般会立即对服务器的SYN+ACK进行ACK应答，因此正常连接会很快建立成功。
- 而异常的连接，不会对服务器的SYN+ACK进行ACK应答，因此异常的连接最终都会堆积到暂存队列当中。

---

### 使用[Wireshark](https://so.csdn.net/so/search?q=Wireshark&spm=1001.2101.3001.7020)分析TCP通信流程

> Wireshark（前称Ethereal）是一个Windows下的网络抓包工具。

在使用Wireshark时可以通过设置过滤器，来抓取满足要求的数据包。

**针对IP进行过滤：**

- 抓取指定源地址的包：`ip.src == 源IP地址`。

- 抓取指定目的地址的包：`ip.dst == 目的IP地址`。

- 抓取源或目的地址满足要求的包：`ip.addr == IP地址`等价于`ip.src == 源IP地址 or ip.dst == 目的IP地址`。

- 抓取除指定IP地址之外的包：`!(表达式)`。

  

**针对协议进行过滤：**

- 抓取指定协议的包：`协议名`（只能小写）。
- 抓取多种指定协议的包：`协议名1 or协议名2`。
- 抓取除指定协议之外的包：`not 协议名`或`!协议名`。



**针对端口进行过滤（以TCP协议为例）：**

- 抓取指定端口的包：`tcp.port == 端口号`。

- 抓取多个指定端口的包：`tcp.port >= 2048`（抓取端口号高于2048的包）。

  

**针对长度和内容进行过滤：**

- 抓取指定长度的包：`udp.length < 30 http.content_length <= 20`。
- 抓取指定内容的包：`http.request.urimatches "指定内容"`。



#### 抓包示例

这里我们抓取指定源IP地址或目的IP地址的数据包。

![img](https://img-blog.csdnimg.cn/23b4b22e1b504217b30c7ec35846bbc4.png)

当我们用telnet命令连接该服务器后，就可以抓取到三次握手时双方交互的数据包。

![img](https://img-blog.csdnimg.cn/0e75ff3e1953455c86ff06b96ec2ef4a.png)

而当我们退出telnet命令后，就可以抓取到四次挥手时双方交互的数据包。（此处四次挥手时进行了捎带应答，第二次挥手和第三次挥手合并在了一起）

![img](https://img-blog.csdnimg.cn/f5c7b675fcc04ec6947d2f4f7211c227.png)

![img](https://img-blog.csdnimg.cn/e87a1423a28346ac9438579f9743281f.png)

---



## 2. TCP与UDP



### TCP与UDP对比



- **TCP协议**

  TCP协议叫做传输控制协议（Transmission Control Protocol），TCP协议是一种面向连接的、可靠的、基于字节流的传输层通信协议。

  TCP协议是面向连接的，如果两台主机之间想要进行数据传输，那么必须要先建立连接，当连接建立成功后才能进行数据传输。其次，TCP协议是保证可靠的协议，数据在传输过程中如果出现了丢包、乱序等情况，TCP协议都有对应的解决方法。



- **UDP协议**

  UDP协议叫做用户数据报协议（User Datagram Protocol），UDP协议是一种无需建立连接的、不可靠的、面向数据报的传输层通信协议。

  使用UDP协议进行通信时无需建立连接，如果两台主机之间想要进行数据传输，那么直接将数据发送给对端主机就行了，但这也就意味着UDP协议是不可靠的，数据在传输过程中如果出现了丢包、乱序等情况，UDP协议本身是不知道的。



- **TCP/UDP对比**

TCP协议虽然是保证可靠性的协议，但不能说TCP就一定比UDP好，因为TCP保证可靠性也就意味着TCP需要做更多的工作，而UDP不保证可靠性也就意味着UDP足够简单。

- TCP常用于可靠传输的情况，应用于文件传输，重要状态更新等场景。
- UDP常用于对高速传输和实时性较高的通信领域，例如早期的QQ、视频传输等。另外UDP可以用于广播。

也就是说，UDP和TCP没有谁最好，只有谁最合适，网络通信时具体采用TCP还是UDP完全取决于上层的应用场景。

---

### 用UDP实现可靠传输（经典面试题）

当面试官让你用UDP实现可靠传输时，你一定要立马想到TCP协议，因为TCP协议就是当前比较完善的保证可靠性的协议，面试官让你用UDP这个不可靠的协议来实现可靠传输，无非就是让你在应用层来实现可靠性，此时就可以参考TCP协议保证可靠性的各种机制。

例如：

- 引入序列号，保证数据按序到达。
- 引入确认应答，确保对端接收到了数据。
- 引入超时重传，如果隔一段时间没有应答，就进行数据重发。
- …

但TCP保证可靠性的机制太多了，当你被面试官问到这个问题时，最好与面试官进一步进行沟通，比如问问这个用UDP实现可靠传输的应用场景是什么。因为TCP保证可靠性的机制太多了，但在某些场景下可能只需要引入TCP的部分机制就行了，因此在不同的应用场景下模拟实现UDP可靠传输的侧重点也是不同的。
