# 1. POSIX信号量



<font color="red" size="3.5">**#问：**</font>**什么是信号量？**

> 1. 共享资源 -> 任何一个时刻都只有一个执行流在进行访问 -> 临界资源、临界区的概念。
>
>    ![image-20240131234144939](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240131234144939.png)



<font color="red" size="3.5">**#问：**</font>**如果一个共享资源，不当做一个整体，而让不同的执行流访问不同的区域的话，那么不就可以继续并发了吗？**

>  是的，当只有访问同一个资源的时候，我们在进行同步或者互斥。
>
> 以此做到大部分依旧是并发，小部分情况下是互斥、同步。这样就可以更大力度的提高线程的效率。
>
> ![image-20240131234339139](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240131234339139.png)
>
> 以此做到大部分依旧是并发，小部分情况下是互斥、同步。这样就可以更大力度的提高线程的效率。







> **根据以上模型：**
>
> <font color="red" size="3.5">**#问：**</font>**怎么知道一共有多少个资源？中途还剩多少资源？**
>
> <font color="red" size="3.5">**#问：**</font>**怎么保证这个资源就是给我们的？我们怎么知道我们一定可以具有一个共享资源？**



**电影院的例子**

> ![image-20240131234714065](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240131234714065.png)
>
> 资源（座位）的预定机制 **——** 代表我们预定了资源（座位），其一定会为我们留这一个资源（座位），别人不会抢，也没法抢，因为持有信号量（票）。

​	

- **信号量的本质：**

  >   是一个计算器，访问临界资源的时候，必须先申请信号量资源（sem--，预定资源，p），使用完毕信号量资源（sem++，释放资源，v）。



<font color="red" size="3.5">**#问：**</font>**我们想申请资源，我们需要先申请信号量（sem--），如果我们申请完信号量，但是我们不去访问呢？**

>  不会出任何问题，对应的信号量资源中一定会有其一个。比如有6个信号，6个线程申请了，5个线程执行，1个不执行。此时再来1个线程（第7个），想申请信号量，不行申请不成功。因为此时信号量已经--到0了，不能再申请了。不访问但资源也会对应的留着。



- **如何理解信号量的使用：**

> 我们申请了一个信号量 -> 当前执行流一定具有一个资源，可以被它使用 -> 是哪一个资源，需要程序员结合场景，自定义编码完成。



- **解决前面的问题：**

  > <font color="red" size="3.5">**#问：**</font>**怎么知道一共有多少个资源？中途还剩多少资源？**
  >
  >  因为信号量初始化的时候初始为几，就代表一共多少资源。还剩多少资源取决于信号量在被使用期间，信号量的值剩几就是几。

  > <font color="red" size="3.5">**#问：**</font>**怎么保证这个资源就是给我们的（程序员编码）？我们怎么知道我们一定可以具有一个共享资源（信号量）？**
  >
  > 需要程序员结合场景，自定义编码完成的。
  >
  > 信号量本质是一个计数器，只要申请成功了，其是对于临界资源的预定机制，只要预定成功就一定会有对应的资源。



---



## 1.1 **sem_init - 初始化信号量**

![image-20240131235803033](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240131235803033.png)

> ```c++
> #include <semaphore.h>
> // 初始化信号量
> int sem_init(sem_t *sem, int pshared, unsigned int value);
> ```
>
> 参数：
>         sem：信号量。
>         pshared：0表示线程间共享，非零表示进程间共享。
>         value：信号量初始值。
> 返回值：
> 		成功时返回0。
> 		出现错误时，返回-1，并设置errno以指示错误。



---



## 1.2 **sem_destroy -** 销毁信号量

![image-20240201000004185](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201000004185.png)

> ```c++
> #include <semaphore.h>
> // 销毁信号量
> int sem_destroy(sem_t *sem);
> ```
>
> 参数：
>
> ​    **sem：**信号量。
>
> 返回值：
>
> - 成功时返回0。
> - 出现错误时，返回-1，并设置errno以指示错误。



---



## 1.3 **sem_wait - 等待信号量**（P操作）

![image-20240201000141049](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201000141049.png)



> ```c++
> // 申请信号量资源，申请成功，继续往后走。申请不成功，默认阻塞。
> int sem_wait(sem_t *sem);
>  
> // 申请信号量资源，申请成功，继续往后走。申请不成功，立马返回(errno设置为EAGAIN)。
> int sem_trywait(sem_t *sem);
>  
> // 指定特定时间abs_timeout，在该时间段内，未申请成功，挂起等待，超过时间成功，立马返回(errno设置为EAGAIN)。
> int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
> ```
>
> ```c++
> #include <semaphore.h>
> // 等待信号量，会将信号量的值--
> int sem_wait(sem_t *sem);
> ```
>
> 参数：
>
> ​    **sem：**信号量。
>
> 返回值：
>
> - 成功时返回0。
> - 出现错误时，返回-1，并设置errno以指示错误。

---



## 1.4 **sem_post - 发布信号量**（V操作）

![image-20240201000348328](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201000348328.png)

> ```c++
> #include <semaphore.h>
> //发布信号量，表示资源使用完毕，可以归还资源了。将信号量值++
> int sem_post(sem_t *sem);
> ```
>
> 参数：
>
> ​    **sem：**信号量。
>
> 返回值：
>
> - 成功时返回0。
> - 出现错误时，返回-1，并设置errno以指示错误。



---



# 2. **基于环形队列的生产消费模型**



## 2.1 **数据结构 - 环形结构**

> 有环形结构的实现，有链表的实现、线性数组的实现等，此处采取线性数组的实现。

![image-20240201000838863](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201000838863.png)

利用线性数组队列的物理结构数据结构，实现逻辑结构上的环形队列。

- **物理结构：**在计算机里，真实的结构的形式存在。
- **逻辑结构：**程序员看待这个结构的方式。



**逻辑结构的意义：**

>   物理结构不便于思考，不便于实现各种逻辑。锁以使用软件封装，将物理结构变为逻辑结构，然后基于逻辑结构展开思考。



对于基于环形队列的生产消费模型重点不是判断是否循环一圈的，if (start == end)判断，而是：

1. **判空**
2. **判满**

![image-20240201001151762](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201001151762.png)

因为if (start == end)无法判断空、满，所以不可使用。所以单纯的生产一个，往后移动一个，这个方法是很不好的。



**环形队列的生产消费模型中，常见的判空、判满的方式有两种：**

> 1. <font color="red" size="3.5">  计数器：</font>
>        0表示空，n表示满。

> 2. <font color="red" size="3.5"> 专门浪费一个格子：</font>
>
>     当前格子的下一个格子，进行index %= n；index == 0；即当前格子不放数据，并且此时判断环形队列。
>
>    
>
>    **为空：**消费与生产在一个格子
>
>    > ![image-20240201002129205](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201002129205.png)
>
>    **为满：**生产的下标+1再%n等于消费。
>
>    > ![image-20240201002212373](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201002212373.png)



---

# 3. **实现原理**

> 这个环形结构，既是生产者的生产结构，也是消费者的消费结构。所以，势必是生产与消费的共享资源。并且是多线程下的共享资源，就势必要考虑多线程访问的线程安全问题，以及多线程之间同步和互斥的问题。如果我们添加使用之前的互斥锁、条件变量的方案，有数据就让消费者消费，环形队列没有满就让环形队列生产。是可以实现多线程协同的，就是因为环形队列是和基于阻塞队列的生产消费模型一样的，是也有空、也有满。所以使用基于阻塞队列的生产消费模型一样的加锁也是可以的。
>



 但是就会有一个问题，如果用加锁方案，潜台词就是：在环形结构当中，如果加锁，我们是将环形结构看作整体使用。





> **需要注意的关键问题：**
>
> > -  **为空：**消费者不能消费，因为对应的环形结构里根本就没有任务。
>
> > - **为满：**生产者不能生产，因为会将环形结构里未被使用的任务覆盖。



 所以，为空的时候是不期望消费者运行的，为满的时候是不期望生产者运行的。



- **从数据结构视角：**就是要么为空，要么为满。

- **从多线程视角：**生产和消费要有互斥或者同步问题。

- 在任意时刻，无论是消费者，还是生产者只能有一个在跑，是互斥的，互斥的运行需要同步的问题。



生产和消费指向同一个位置是小概率事件，大概率生产和消费都指向的是不同的位置。

- 想让当生产和消费指向同一个位置，<font color="red" size="3.5"> 具有互斥、同步关系就可以了。</font>

- 想让当生产和消费不指向同一个位置，让它们<font color="red" size="3.5"> 并发执行。</font>



 **期望：**

- 生产者不能将消费者套圈。 因为在环行结构中套圈是错误的。
- 消费者不能超过生产者，永远就相当于一个跟随者的方式。



**实现：**

- **为空：**一定要让生产者先运行。
- **为满：**一定要让消费者先运行。
- **其他情况：**可以并发访问。





<font color="red" size="3.5"> **如何引入信号量？**</font>

>  信号量是用来描述临界区中临界资源的一个计数器。所以在信号量的视角，就是将生产者与消费者最关心的资源，来进行计数，来使用信号量描述资源数量。因为为空、为满就是判断资源数目。

- **生产者：**最关注的是环形结构中的空间资源（有没有空间放数据）**—>**（信号量）spaceSem **—>** 起始N。
- **消费者：**最关注的是环形结构中的数据资源（有没有数据可消费）**—>**（信号量）dataSem  **—>** 起始0。



**操作：**

> <font color="red" size="3.5"> **生产：**</font>
>
> - 一个生产者想生产，生产就需要有空间，就需要先进行对空间预约，即申请信号量（P操作）。
>
>   ![image-20240201004742673](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201004742673.png)
>
>   - 生产者将数据生产到环形队列的特定位置，生产者将数据生产了，将数据放入进去了。于是，生产者去生产下一个位置了，但是当前位置是依旧被占用的。所以生产者不能归还空间资源，于是V的是dataSam。

> <font color="red" size="3.5"> **消费：**</font>
>
> - 一个消费者想消费，消费就需要有数据，就需要先进行对数据预约，即申请信号量（P操作）。
>
>   ![image-20240201005218379](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201005218379.png)
>
>   - 消费者将数据数据拿走了，于是这个数据曾经占用的空间资源就空出来了，于是V的是<font color="red" size="3.5"> spaceSam。</font>





<font color="red" size="3.5"> **问 :: **</font>**当生产者与消费者同时在一个位置的时候，如何保证的谁先执行？**

>   **以开始为例：**生产者与消费者同时进行运行的时候，一定要先进行初始化信号量，于是spaceSem为N，dataSem为0。于是接着同时进行运行，就各自执行P操作，消费者一看dataSem为0申请不出来，于是申请失败，消费线程直接被挂起。只能等生产者先生产。
>
> - **为空：**spaceSem为N，dataSem为0。只有生产者能执行。
> - **为满：**spaceSem为0，dataSem为N。只有消费者能执行。



 以此，保证在同一个位置的时候，生产和消费的步调是互斥、同步的。并且也以此保证了：

- 生产者永远一定不会将消费者套圈。
- 消费者永远一定不会超过生产者，永远就相当于一个跟随者的方式。



 信号量，就帮我们解决了，这一系列的问题。

---

## 3.1 代码实现  单生产 - 单消费

### **Sem.hpp**

> 封装的信号量。
>
> ```c++
> #ifndef _SEM_HPP_
> #define _SEM_HPP_
>  
> #include <iostream>
> #include <semaphore.h>
>  
> class Sem
> {
> public:
>     Sem(int value)
>     {
>         sem_init(&sem_, 0, value);
>     }
>  
>     ~Sem()
>     {
>         sem_destroy(&sem_);
>     }
>  
>     void p()
>     {
>         sem_wait(&sem_);
>     }
>  
>     void v()
>     {
>         sem_post(&sem_);
>     }
>  
> private:
>     sem_t sem_;
> };
>  
> #endif
> ```



---

### **ringQueue.hpp**

```c++
#ifndef _Ring_QUEUE_HPP_
#define _Ring_QUEUE_HPP_
 
#include <iostream>
#include <vector>
#include <pthread.h>
#include "Sem.hpp"
 
const int g_dafault_num = 5;
 
template <class T>
class RingQueue
{
public:
    RingQueue(const int default_num = g_dafault_num)
        : ring_queue_(default_num)
        , num_(default_num)
        , p_step_(0)
        , c_step_(0)
        , space_sem_(default_num)
        , data_sem_(0)
    {}
    
    ~RingQueue()
    {}
 
    // 生产者: 空间资源
    void push(const T &in)
    {
        // 申请空间资源 - 空间少一个
        space_sem_.p();
 
        // 100%拿到了空间资源
        ring_queue_[p_step_++] = in;
        p_step_ %= num_;
 
        // 使用后空间后，数据多一个
        data_sem_.v();
    }
 
    // 消费者：数据资源
    void pop(T *out)
    {
        // 申请胡数据资源 - 数据少一个
        data_sem_.p();
 
        // 100%拿到了数据资源
        *out = ring_queue_[c_step_++];
        c_step_ %= num_;
 
        // 使用后数据后，空间多一个
        space_sem_.v();
    }
 
private:
    std::vector<T> ring_queue_;
    int num_;
    int c_step_; // 消费下标
    int p_step_; // 生产下标
    Sem space_sem_;
    Sem data_sem_;
};
 
#endif
```

---



### **testWain.cc**

```c++
#include "ringQueue.hpp"
#include <iostream>
#include <ctime>
#include <cstdlib>
 
//消费者
void *consumer(void *args)
{
    RingQueue<int>* rq = (RingQueue<int>*)args;
    while(true)
    {
        int x;
        // 1. 从环形队列中获取任务或者数据
        rq->pop(&x);
 
        // 2. 进行一定的处理 -- 不要忽略它的时间消耗问题
        std::cout << "消费：" << x << std::endl;
    }
    return nullptr;
}
 
// 生产者
void *producer(void *args)
{
    RingQueue<int>* rq = (RingQueue<int>*)args;
    while(true)
    {
        int x = rand()%100 + 1;
        // 1. 构建数据或者任务对象 -- 一般是可以从外部来 -- 不要忽略它的时间消耗问题
        rq->push(x);
 
        // 2. 推送到环形队列中
        std::cout << "生产：" << x << std::endl;
    }
    return nullptr;
}
 
int main()
{
    srand((uint64_t)time(nullptr));
    RingQueue<int> *rq = new RingQueue<int>();
    pthread_t c, p;
    pthread_create(&c, nullptr, consumer, (void*)rq);
    pthread_create(&p, nullptr, producer, (void*)rq);
 
    pthread_join(c, nullptr);
    pthread_join(p, nullptr);
    return 0;
}
```

---



![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c82b5fe45ae84bd6aa04808ab53c6b34.gif#pic_center)

 

我们可以通过将消费放慢，于是可以看到生产者瞬间生产满，然后由于信号量space_sum_ = 0；所以无法申请到空间资源，于是阻塞等待。然后就是消费一个，就有空间了，于是立马又生产一个，然后又阻塞等待消费者消费。

![image-20240201135454155](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201135454155.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/abc8311992744982a1ddad88e3dfa2c2.gif#pic_center)

![image-20240201135715669](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201135715669.png)

 对于单生产、单消费，由于有信号量的存在，所以对于同一个位置的生产、消费，不用担心他们会对同一个位置并发访问。为满，空间资源不就绪。为空，数据资源不就绪。所以一定有一方竞争失败，根本不用担心并发访问。





<font color="red" size="3.5"> **问**::  </font>**如何在当前的代码下实现多生产，多消费？** 

>  这个时候我们在一个关系：生产者与消费者，的关系上新增了两个：生产者与生产者、消费者与消费者。
>
> - **生产者与生产者：**竞争关系，互斥关系。
> - **消费者与消费者：**竞争关系，互斥关系。
>
> 以，对于此的解决方式，是势必要使用加锁的。加两把锁：生产者与生产者一把、消费者与消费者一把。



<font color="red" size="3.5"> **问:: ** </font>**生产者们的临界资源是什么？消费者们的临界资源是什么？**

>  我们将环形队列对应的拆做了很多个小格子，**生产者们**与**消费者们**都是竞争的这个小格子。而这个小格子的空间使用下标来标识。所以它们需要保护的是下标。
>
> ![image-20240201140912127](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201140912127.png)





但是，要知道，加锁的区域是越小越好，而信号量是资源的预定机制，并且还一定是安全的（具有原子性）。那么：

<font color="red" size="3.5">**#问：**</font>**先加锁，还是先申请信号量？**

>   **先申请信号量！**
>
> ![image-20240201154257637](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201154257637.png)
>
>    因为，就相当于如果我们先加锁，如此去申请信号量的线程一定是很少的，而这个程序的工作效率是高还是低，是取决于我们将这个资源如何快速的派发给线程。
>
> - **如果先加锁：**
>
>   - 先申请锁，然后再申请信号量。然后全部做完、跑完了才能让下一个线程进来。
>
> - **如果先申请信号量：**
>
>   - 先申请信号量（先分资源的预定），最后哪怕只有一个线程进入到临界区中。虽然其他线程没有进入临界资源，但是可以并发的去竞争信号量。
>
>   > **就如同电影院：**
>   >
>   > - 先到网上，每个人可以同时的去抢票，到时候直接看。
>   > - 看的时候，到电影院排队，然后买票，再看。

---



## 3.2 代码实现 多生产 多消费

### **Sem.hpp**

```c++
#ifndef _SEM_HPP_
#define _SEM_HPP_
 
#include <iostream>
#include <semaphore.h>
 
class Sem
{
public:
    Sem(int value)
    {
        sem_init(&sem_, 0, value);
    }
 
    ~Sem()
    {
        sem_destroy(&sem_);
    }
 
    void p()
    {
        sem_wait(&sem_);
    }
 
    void v()
    {
        sem_post(&sem_);
    }
 
private:
    sem_t sem_;
};
 
#endif
```

---

### **ringQueue.hpp**

```c++
#ifndef _Ring_QUEUE_HPP_
#define _Ring_QUEUE_HPP_
 
#include <iostream>
#include <vector>
#include <pthread.h>
#include "Sem.hpp"
 
const int g_dafault_num = 5;
 
template <class T>
class RingQueue
{
public:
    RingQueue(const int default_num = g_dafault_num)
        : ring_queue_(default_num), num_(default_num), p_step_(0), c_step_(0), space_sem_(default_num), data_sem_(0)
    {
        pthread_mutex_init(&c_lock_, nullptr);
        pthread_mutex_init(&p_lock_, nullptr);
    }
 
    ~RingQueue()
    {
        pthread_mutex_destroy(&c_lock_);
        pthread_mutex_destroy(&p_lock_);
    }
 
    // 生产者: 空间资源
    void push(const T &in)
    {
        // 申请空间资源 - 空间少一个
        space_sem_.p();
 
        // 加锁
        pthread_mutex_lock(&p_lock_); // 一定是竞争成功的生产者线程 -- 就一个！
 
        // 内部一定是单线程
        // 100%拿到了空间资源
        ring_queue_[p_step_++] = in;
        p_step_ %= num_;
 
        // 释放锁
        pthread_mutex_unlock(&p_lock_);
 
        // 使用后空间后，数据多一个
        data_sem_.v();
    }
 
    // 消费者：数据资源
    void pop(T *out)
    {
        // 申请胡数据资源 - 数据少一个
        data_sem_.p();
 
        // 加锁
        pthread_mutex_lock(&c_lock_); // 一定是竞争成功的消费者线程 -- 就一个！
 
        // 内部一定是单线程
        // 100%拿到了数据资源
        *out = ring_queue_[c_step_++];
        c_step_ %= num_;
 
        // 释放锁
        pthread_mutex_unlock(&c_lock_);
 
        // 使用后数据后，空间多一个
        space_sem_.v();
    }
 
private:
    std::vector<T> ring_queue_;
    int num_;
    int c_step_; // 消费下标
    int p_step_; // 生产下标
    Sem space_sem_;
    Sem data_sem_;
    pthread_mutex_t c_lock_; // 消费者的锁
    pthread_mutex_t p_lock_; // 生产者的锁
};
 
#endif
```

---

### **testMain.cc**

```c++
#include "ringQueue.hpp"
#include <iostream>
#include <ctime>
#include <cstdlib>
#include <unistd.h>
 
//消费者
void *consumer(void *args)
{
    RingQueue<int>* rq = (RingQueue<int>*)args;
    while(true)
    {
        int x;
        // 1. 从环形队列中获取任务或者数据
        rq->pop(&x);
 
        // 2. 进行一定的处理 -- 不要忽略它的时间消耗问题
        std::cout << "消费: " << x << " [" << pthread_self() << "]" << std::endl;
    }
    return nullptr;
}
 
// 生产者
void *producer(void *args)
{
    RingQueue<int>* rq = (RingQueue<int>*)args;
    while(true)
    {
        int x = rand()%100 + 1;
        // 1. 构建数据或者任务对象 -- 一般是可以从外部来 -- 不要忽略它的时间消耗问题
        rq->push(x);
 
        // 2. 推送到环形队列中
        std::cout << "生产: " << x << " [" << pthread_self() << "]" << std::endl;
    }
    return nullptr;
}
 
int main()
{
    srand((uint64_t)time(nullptr));
    RingQueue<int> *rq = new RingQueue<int>();
    pthread_t c[3],p[2];
    pthread_create(c, nullptr, consumer, (void*)rq);
    pthread_create(c+1, nullptr, consumer, (void*)rq);
    pthread_create(c+2, nullptr, consumer, (void*)rq);
 
    pthread_create(p, nullptr, producer, (void*)rq);
    pthread_create(p+1, nullptr, producer, (void*)rq);
 
    for(int i = 0; i < 3; i++) pthread_join(c[i], nullptr);
    for(int i = 0; i < 2; i++) pthread_join(p[i], nullptr);
    return 0;
}
```

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/da3acdc9820648d7acdf2b53dc290309.gif#pic_center)

![image-20240201162207524](C:\Users\。\AppData\Roaming\Typora\typora-user-images\image-20240201162207524.png)

此时我们看见是几个线程在同时生成和消费

---



<font color="red" size="3.5">**#问：**</font>**多生产多消费的意义在哪里？**

>  是不是因为加锁，所以真正生产和真正消费也就只有一个线程？我们不能，也不要狭隘的认为，把任务或者数据放在交易场所，就是生产和消费了。我们将数据或者任务生产前和拿到之后处理，才是最耗费时间的。

- **生产的本质：**私有的任务-> 公共空间中
- **消费的本质：**公共空间中的任务-> 私有的



虽然，生产任务、拿任务，都是一个一个的做的，但是处理任务的时候，是可以变为并发的。并发的生产数据，并发的处理数据。

>    就像食堂的多个窗口：不是阿姨打菜、我们打饭然后就完了，而是阿姨做菜的时候，和我们吃饭的时候才是最耗费时间的。





<font color="red" size="3.5">**#问：**</font>**信号量本质是一把计数器 -> 计数器的意义是什么？**

>  计数器是用来表示，临界资源中的特定资源。在阻塞队列的生产者消费者模型中：申请锁 -> 判断与访问 -> 释放锁 -> 本质是我们并不清楚临界资源的情况！但是信号量是提前让程序员初始化好的计数器，也就是说：信号量要提前预设资源的情况，而且在pv变化过程中，我们可以在外部就能知晓临界资源的情况！
>



   <font color="red" size="3.5">**计数器的意义：**</font>可以不用进入临界区，就可以得知资源情况，甚至可以减少临界区内部的判断！



> 在没有信号量这个计数器的时候，因为不知道临界资源的状态，所以需要先加锁，再使用临界资源进行判断（条件变量），于是满了就释放锁并挂起。而有了信号量，因为信号量是资源的预定机制，其就是用来表明，环形队列中的情况。（在外部就可以得知临界区的状况）