# list反向迭代器

# list的反向迭代器

-   在C++ STL中，迭代器是一种指针，用于遍历容器内的元素。迭代器一般分为正向迭代器和反向迭代器。正向迭代器从容器的起始位置开始遍历到末尾位置，而反向迭代器则从末尾位置开始遍历到容器的起始位置。
-   首先，一个反向迭代器 `__reverse_iterator` 需要保存一个正向迭代器 `Iterator`。

```c++
template <class Iterator,class Ref,class Ptr>
struct __reverse_iterator
{
    Iterator cur;    // 当前的正向迭代器
};
```

-   对于反向迭代器的 `++` 和 `--` 操作，需要调用正向迭代器的 `--` 和 `++` 操作，让其从后向前遍历容器中的元素。

```c++
Riterator operator++()
{
    --cur;    // 调用正向迭代器的 -- 操作，向前移动一位
    return *this;
}

Riterator operator++(int)
{
    // 同上，使用后置 ++ 操作符时需要返回一个临时对象
    Riterator it(*this);
    --cur;
    return it;
}

Riterator operator--()
{
    ++cur;    // 调用正向迭代器的 ++ 操作，向后移动一位
    return *this;
}

Riterator operator--()
{
    Riterator it(*this);
    ++cur;
    return it;
}
```

-   对于反向迭代器的 `*` 操作，其要返回迭代器逆序访问到的元素的引用，即 `--cur` 对应的元素的引用。

```c++
Ref operator*()
{
    auto tmp = cur;    // 保存一份正向迭代器，方便后面 -- 操作
    --tmp;            // 正向迭代器往前移一位，即反向迭代器往后移一位
    return *tmp;      // 返回移动之后的元素的引用
}

Ptr operator- >()
{
    return &(operator*());
}
```

-   最后，需要对 `!=` 和 `==` 操作符进行重载，以实现反向迭代器的比较。

```c++
bool operator!=(const Riterator&  it)
{
    return cur != it.cur;
}

bool operator==(const Riterator& it)
{
    return cur == it.cur;
}
```

-   以下便是反向迭代器的实现原理，通过倒序访问容器中的元素使得反向迭代器能够比较高效地访问容器，提高代码的可读性和性能。

```c++
#pragma once

namespace Xm
{
    template <class Iterator,class Ref,class Ptr>
    struct __reverse_iterator
    {
        Iterator _cur;
        typedef __reverse_iterator<Iterator, Ref, Ptr> Riterator;

        __reverse_iterator(Iterator it)
            :_cur(it)
        {}

        // 前置++运算符重载
        Riterator operator++()
        {
            --_cur;
            return *this;
        }

        // 后置++运算符重载
        Riterator operator++(int)
        {
            Riterator it(*this);
            --_cur;
            return it;
        }

        // 前置--运算符重载
        Riterator operator--()
        {
            ++_cur;
            return *this;
        }

        // 后置--运算符重载
        Riterator operator--(int)
        {
            Riterator it(*this);
            ++_cur;
            return it;
        }

        // *运算符重载
        Ref operator*()
        {
            auto tmp = _cur;
            --tmp;
            return *tmp;
        }

        // ->运算符重载
        Ptr operator->()
        {
            return &(operator*());
        }

        // !=运算符重载
        bool operator!=(const Riterator&  it)
        {
            return _cur != it._cur;
        }

        // ==运算符重载
        bool operator==(const Riterator& it)
        {
            return _cur == it._cur;
        }
    };
}

```

-   疑问点: 为什么不会调用方向迭代器自己的++和—

> 在反向迭代器的实现中，`operator--()` 和 `operator++()` 需要调用正向迭代器的 `--` 和 `++` 操作，以实现反向遍历容器的效果。以 `operator--()` 为例，反向迭代器的实现如下：

```c++
Riterator operator--()
{
    ++cur;    // 调用正向迭代器的 ++ 操作，即容器中的下一个元素
    return *this;
}
```

> 这里是通过调用正向迭代器的 `operator++()` 实现对容器进行反向遍历。在这个过程中，并没有调用到该反向迭代器自身定义的 `operator--()`，因此也不存在递归调用的情况。如果反向迭代器自身重载了 `operator--()`，且在函数体内部继续调用自身的 `operator--()`，就会导致无限循环的情况。

```c++
Riterator operator--()
{
    // 调用自身的 operator--()，但是由于反向遍历，这个调用永远不会结束
    this->operator--();
    return *this;
}
```

-   因此，反向迭代器的实现不会调用自身的 `operator--()` 或 `operator++()`，而是直接调用正向迭代器的 `--` 和 `++` 操作实现反向遍历效果。
