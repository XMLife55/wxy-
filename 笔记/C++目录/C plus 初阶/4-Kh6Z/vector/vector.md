# vector

## 目录

-   [vector 空间增长问题](#vector-空间增长问题)
-   [vector 迭代器失效问题。（重点）](#vector-迭代器失效问题重点)

## vector 空间增长问题

-   capacity的代码在vs和g++下分别运行会发现，vs下capacity是按1.5倍增长的，g++是按2倍增长的。这个问题经常会考察，不要固化的认为，vector增容都是2倍，具体增长多少是根据具体的需求定义的。vs是PJ版本STL，g++是SGI版本STL。
-   reserve只负责开辟空间，如果确定知道需要用多少空间，reserve可以缓解vector增容的代价缺陷问

题。

-   resize在开空间的同时还会进行初始化，影响size。

```c++
// 测试vector的默认扩容机制
void TestVectorExpand()
{
  size_t sz;
  vector<int> v;
  sz = v.capacity();
  cout << "making v grow:\n";
  for (int i = 0; i < 100; ++i)
  {
    v.push_back(i);
    if (sz != v.capacity())
    {
      sz = v.capacity();
      cout << "capacity changed: " << sz << '\n';
    }
  }
}
vs：运行结果：vs下使用的STL基本是按照1.5倍方式扩容
making foo grow:
capacity changed: 1
capacity changed: 2
capacity changed: 3
capacity changed: 4
capacity changed: 6
capacity changed: 9
capacity changed: 13
capacity changed: 19
capacity changed: 28
capacity changed: 42
capacity changed: 63
capacity changed: 94
capacity changed: 141

g++运行结果：linux下使用的STL基本是按照2倍方式扩容
making foo grow:
capacity changed: 1
capacity changed: 2
capacity changed: 4
capacity changed: 8
capacity changed: 16
capacity changed: 32
capacity changed: 64
capacity changed: 128

```

```c++
// 如果已经确定vector中要存储元素大概个数，可以提前将空间设置足够
// 就可以避免边插入边扩容导致效率低下的问题了
void TestVectorExpandOP()
{
  vector<int> v;
  size_t sz = v.capacity();
  v.reserve(100); // 提前将容量设置好，可以避免一遍插入一遍扩容
  cout << "making bar grow:\n";
  for (int i = 0; i < 100; ++i)
  {
    v.push_back(i);
    if (sz != v.capacity())
    {
      sz = v.capacity();
      cout << "capacity changed: " << sz << '\n';
    }
  }
}
```

## vector 迭代器失效问题。（重点）

-   迭代器的主要作用就是让算法能够不用关心底层数据结构，其底层实际就是一个指针，或者是对指针进行了封装，比如：vector的迭代器就是原生态指针T\* 。因此迭代器失效，实际就是迭代器底层对应指针所指向的空间被销毁了，而使用一块已经被释放的空间，造成的后果是程序崩溃(即如果继续使用已经失效的迭代器，程序可能会崩溃)。

> 对于vector可能会导致其迭代器失效的操作有：

1.  会引起其底层空间改变的操作，都有可能是迭代器失效，比如：resize、reserve、insert、assign、push\_back等。

```c++
#include <iostream>
using namespace std;
#include <vector>
int main()
{
  vector<int> v{ 1,2,3,4,5,6 };

  auto it = v.begin();

  // 将有效元素个数增加到100个，多出的位置使用8填充，操作期间底层会扩容
  // v.resize(100, 8);

  // reserve的作用就是改变扩容大小但不改变有效元素个数，操作期间可能会引起底层容量改变
  // v.reserve(100);

  // 插入元素期间，可能会引起扩容，而导致原空间被释放
  // v.insert(v.begin(), 0);
  // v.push_back(8);

  // 给vector重新赋值，可能会引起底层容量改变
  v.assign(100, 8);

  /*
  出错原因：以上操作，都有可能会导致vector扩容，也就是说vector底层原理旧空间被释放掉，
   而在打印时，it还使用的是释放之间的旧空间，在对it迭代器操作时，实际操作的是一块已经被释放的
   空间，而引起代码运行时崩溃。
  解决方式：在以上操作完成之后，如果想要继续通过迭代器操作vector中的元素，只需给it重新
   赋值即可。
  */
  while (it != v.end())
  {
    cout << *it << " ";
    ++it;
  }
  cout << endl;
  return 0;
}
```

1.  指定位置元素的删除操作--erase

-   stl 规定erase返回删除位置下一个位置迭代器

```c++
#include <iostream>
using namespace std;
#include <vector>
int main()
{
  int a[] = { 1, 2, 3, 4 };
  vector<int> v(a, a + sizeof(a) / sizeof(int));
  // 使用find查找3所在位置的iterator
  vector<int>::iterator pos = find(v.begin(), v.end(), 3);
  // 删除pos位置的数据，导致pos迭代器失效。
  v.erase(pos);
  cout << *pos << endl; // 此处会导致非法访问
  return 0;
}

 erase删除pos位置元素后，pos位置之后的元素会往前搬移，没有导致底层空间的改变，理论上讲迭代
器不应该会失效，但是：如果pos刚好是最后一个元素，删完之后pos刚好是end的位置，而end位置是
没有元素的，那么pos就失效了。因此删除vector中任意位置上元素时，vs就认为该位置迭代器失效
了。
```

1.  以下代码的功能是删除vector中所有的偶数，请问那个代码是正确的，为什么？

```c++
#include <iostream>
using namespace std;
#include <vector>
//代码错误
int main()
{
  vector<int> v{ 1, 2, 3, 4 };
  auto it = v.begin();
  while (it != v.end())
  {
    if (*it % 2 == 0)
      v.erase(it);
    ++it;
  }

  return 0;
}

//代码正确
int main()
{
  vector<int> v{ 1, 2, 3, 4 };
  auto it = v.begin();
  while (it != v.end())
  {
    if (*it % 2 == 0)
      it = v.erase(it);
    else
      ++it;
  }
  return 0;
}

```

> 因为erase删除后返回下一个元素的迭代器,如果代码1每次都++，则会跳过两次。

> 比如1 2 3 4 5 删除2后，迭代器指向3, 再次++迭代器 指向4，删除4后，迭代器指向5，++迭代器循环结束，但是这只是巧合.我们看看下面的数据.

> **1 2 3 4** 删除2后，迭代器指向3，++迭代器指向4，删除4后迭代器指向最后一个元素v.end(),但是最后++it了导致错过了，所以导致迭代器失效.

> 1 2 4 3 4 5 删除2后，迭代器指向4，迭代器再次++指向3，错过了4。迭代器失效.

> 解决方法：在使用前，对迭代器重新赋值即可

***

1.  &#x20;注意：Linux下，g++编译器对迭代器失效的检测并不是非常严格，处理也没有vs下极端。

```c++

// 1. 扩容之后，迭代器已经失效了，程序虽然可以运行，但是运行结果已经不对了
int main()
{
  vector<int> v{ 1,2,3,4,5 };
  for (size_t i = 0; i < v.size(); ++i)
    cout << v[i] << " ";
  cout << endl;
  auto it = v.begin();
  cout << "扩容之前，vector的容量为: " << v.capacity() << endl;
  // 通过reserve将底层空间设置为100，目的是为了让vector的迭代器失效 
  v.reserve(100);
  cout << "扩容之后，vector的容量为: " << v.capacity() << endl;

  // 经过上述reserve之后，it迭代器肯定会失效，在vs下程序就直接崩溃了，但是linux下不会
  // 虽然可能运行，但是输出的结果是不对的
  while (it != v.end())
  {
    cout << *it << " ";
    ++it;
  }
  cout << endl;
  return 0;
}
程序输出：
1 2 3 4 5
扩容之前，vector的容量为: 5
扩容之后，vector的容量为 : 100
0 2 3 4 5 409 1 2 3 4 5



// 2. erase删除任意位置代码后，linux下迭代器并没有失效
// 因为空间还是原来的空间，后序元素往前搬移了，it的位置还是有效的
#include <vector>
#include <algorithm>
int main()
{
  vector<int> v{ 1,2,3,4,5 };
  vector<int>::iterator it = find(v.begin(), v.end(), 3);
  v.erase(it);
  cout << *it << endl;
  while (it != v.end())
  {
    cout << *it << " ";
    ++it;
  }
  cout << endl;
  return 0;
}
程序可以正常运行，并打印：
4
4 5



// 3: erase删除的迭代器如果是最后一个元素，删除之后it已经超过end
// 此时迭代器是无效的，++it导致程序崩溃
int main()
{
  vector<int> v{ 1,2,3,4,5 };
  // vector<int> v{1,2,3,4,5,6};
  auto it = v.begin();
  while (it != v.end())
  {
    if (*it % 2 == 0)
      v.erase(it);
    ++it;
  }
  for (auto e : v)
    cout << e << " ";
  cout << endl;
  return 0;
}

```

> 从上述三个例子中可以看到：SGI STL中，迭代器失效后，代码并不一定会崩溃，但是运行结果肯定不对，如果it不在begin和end范围内，肯定会崩溃的

1.  与vector类似，string在插入+扩容操作+erase之后，迭代器也会失效

```c++

#include <string>
void TestString()
{
  string s("hello");
  auto it = s.begin();
  // 放开之后代码会崩溃，因为resize到20会string会进行扩容
  // 扩容之后，it指向之前旧空间已经被释放了，该迭代器就失效了
  // 后序打印时，再访问it指向的空间程序就会崩溃
  //s.resize(20, '!');
  while (it != s.end())
  {
    cout << *it;
    ++it;
  }
  cout << endl;
  it = s.begin();
  while (it != s.end())
  {
    it = s.erase(it);
    // 按照下面方式写，运行时程序会崩溃，因为erase(it)之后
    // it位置的迭代器就失效了
    // s.erase(it); 
    ++it;
  }
}
```

-   迭代器失效解决办法：在使用前，对迭代器重新赋值即可

***
