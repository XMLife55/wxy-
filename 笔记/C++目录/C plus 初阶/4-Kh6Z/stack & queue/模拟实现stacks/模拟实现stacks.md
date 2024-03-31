# 模拟实现stacks

-   从栈的接口中可以看出，栈实际是一种特殊的vector，因此使用vector完全可以模拟实现stack。

```c++
namespace bite
{
  template<class T>
  class stack
  {
  public:
    stack() {}
    void push(const T& x) 
    { 
      _c.push_back(x); 
    }
    void pop() 
    { 
      _c.pop_back(); 
    }
    T& top() 
    { 
      return _c.back(); 
    }
    const T& top()const 
    { 
      return _c.back();
    }
    size_t size()const 
    { 
      return _c.size(); 
    }
    bool empty()const 
    { 
      return _c.empty(); 
    }
  private:
    std::vector<T> _c;
  };
}
```

-   底层实现deque\<T>

```c++
/*
 * @file stack.hpp
 * @brief 实现一个通用的栈（堆栈）容器类模板，使用deque或者其他容器作为底层结构存储栈中的元素
 * @author [你的名字]
 * @date [编写日期]
 */

namespace Xm
{
    /**
     * @brief 栈（堆栈）容器类模板
     *
     * @tparam T 栈中元素的类型
     * @tparam Container 底层容器类型，默认为std::deque<T>
     */
    template <class T, class Container = std::deque<T>>
    class stack
    {
    public:
        /**
         * @brief 向栈中压入一个元素
         *
         * @param val 要压入的元素
         */
        void push(const T& val)
        {
            _con.push_back(val);
        }

        /**
         * @brief 从栈中弹出一个元素
         *
         */
        void pop()
        {
            _con.pop_back();
        }

        /**
         * @brief 判断栈是否为空
         *
         * @return 若栈为空，返回true，否则返回false
         */
        bool empty() const
        {
            return _con.empty();
        }

        /**
         * @brief 返回栈顶元素的引用
         *
         * @return 栈顶元素的引用
         */
        T& top()
        {
            return _con.back();
        }

        /**
         * @brief 返回栈顶元素的常引用
         *
         * @return 栈顶元素的常引用
         */
        const T& top() const
        {
            return _con.back();
        }

        /**
         * @brief 返回栈中元素的个数
         *
         * @return 栈中元素的个数
         */
        size_t size() const
        {
            return _con.size();
        }

    private:
        Container _con;  ///< 用于存储栈中元素的底层容器
    };
}
```
