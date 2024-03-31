# 模拟实现Queue

-   借助deque来模拟实现queue，

```c++
#include <deque> // deque 头文件，用于提供双端队列的实现

namespace Xm {
    template<class T, class Container = std::deque<T>>
    class queue
    {
    public:
        void push(const T& val) // 入队操作，将val添加到队列的末尾
        {
            _con.push_back(val); // 在容器的末尾添加一个元素
        }

        void pop() // 出队操作，弹出队列的首元素
        {
            _con.pop_front(); // 弹出位于容器前端的一个元素
        }

        T& front() // 返回队列的首元素，并允许修改
        {
            return _con.front(); // 返回容器前端的一个元素
        }

        const T& front() const // 返回队列的首元素，不允许修改
        {
            return _con.front(); // 返回容器前端的一个元素
        }

        size_t size() const // 返回队列中的元素数量
        {
            return _con.size(); // 返回容器中元素的个数
        }

        T& back() // 返回队列的尾元素，并允许修改
        {
            return _con.back(); // 返回容器末尾的一个元素
        }

        const T& back() const // 返回队列的尾元素，不允许修改
        {
            return _con.back(); // 返回容器末尾的一个元素
        }

        bool empty() const // 判断队列是否为空
        {
            return _con.empty(); // 判断容器是否为空
        }
        
    private:
        Container _con; // 内部使用一个指定类型的容器作为队列的实现
    };
}
```

-   因为queue的接口中存在头删和尾插，因此使用vector来封装效率太低，故可以借助list来模拟实现queue，

具体如下：

```c++
#include<list>

namespace Xm
{
    // 定义了一个队列类，模板类型为 T
    template <class T>
    class queue
    {
    public:
        // 构造函数，初始化列表为空
        queue()
        {}

        // 向队列尾部添加一个元素
        void push(const T& val)
        {
            _con.push_back(val);
        }

        // 弹出队首元素
        void pop()
        {
            _con.pop_front();
        }

        // 返回队首元素的引用
        T& front()
        {
            return _con.front();
        }

        // 返回队首元素的常量引用
        const T& front() const
        {
            return _con.front();
        }

        // 返回队尾元素的引用
        T& back()
        {
            return _con.front();
        }

        // 返回队尾元素的常量引用
        const T& back() const
        {
            return _con.front();
        }

        // 返回队列中元素的个数
        size_t size() const
        {
            return _con.size();
        }

        // 判断队列是否为空
        bool empty()
        {
            return _con.empty();
        }
        
    private:
        // 基于双向链表实现队列
        std::list<T> _con;
    };
}
```
