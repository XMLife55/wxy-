# **static**成员

>    有时候类需要它的一些成员与类本身直接相关，而不是与类的各个对象保持关联。从实现效率的角度来看，没必要每个对象都进行存储。是每个类对象所共享，不属于某个具体的对象。

>   通过在成员之前加上关键字static使得其可以与类关联，和其他成员一样，其可以是public或private的。声明为static的类成员称为类的静态成员，用static修饰的成员变量（常量、引用、指针、类类型等），称之为静态成员变量；用static修饰的成员函数，称之为静态成员函数。静态成员变量一定要在类外进行初始化。
>



**一、静态成员为所有类对象所共享，不属于某个具体的对象，存放在静态区。**

> ```c++
> class A
> {
> private:
> 	static int _a;
> };
> int main()
> {
> 	cout << sizeof(A) << endl; //输出为1，因为静态成员_a是存储在静态区，并为存储与类的空间中
> 	return 0;
> }
> ```

 计算类的大小或是类对象的大小时，静态成员存储于静态区，是属于所有类的同类类型的，是不计入总大小之和的。



**二、静态成员变量必须在类外定义，定义时不添加static关键字，类中只是声明。**

```c++
class A
{
private:
	static int _a; //声明
};
int A::_a = 10; //静态成员变量一定要在类外进行初始化 -- 添加static -> 即error
```

初始化方式

> 数据类型 + 作用域 + 类内的静态成员变量.
>
> (数据类型) (作用域) :: （类内的静态成员变量）  = 





**三、类静态成员即可用 类名::静态成员 或者 对象.静态成员 来访问。**

> ```c++
> class A
> {
> public:	//需要在公有的情况下，类外才可以使用类中的成员变量
> 	static int _a;
> };
> int A::_a = 10; 
> int main()
> {
> 	A a;
> 	cout << a._a << endl;   //类对象.进行访问
> 	cout << A()._a << endl; //匿名对象.进行访问
> 	cout << A::_a << endl;  //类名::进行访问
> 	return 0;
> }
> ```



**四、静态成员也是类的成员，受public、protected、private 访问限定符的限制。**

> ```c++
> class A
> {
> public:
> 	static int _a1;
> private: //protected:
> 	static int _a2;
> };
>  
> int A::_a1 = 10;
> int A::_a2 = 10;
>  
> int main()
> {
> 	cout << A::_a1 << endl; //正确
> 	cout << A::_a2 << endl; //error：无法访问
> 	return 0;
> }
> ```



---

#  static**静态成员函数**

  静态成员函数主要为了调用方便，不需要生成对象就能调用。 静态成员函数的作用类似于：一个在命名空间中的全局函数。

> ```c++
> class A
> {
> public:
> 	static void Func1()
> 	{
> 		Func2(); //没有this指针，无法准换为this->Fun2();
> 	}
>  
> 	void Func2()
> 	{
> 		cout << "void Func2()" << endl;
> 	}
> };
>  
> int _a = 10;
> ```





**一、静态成员函数没有隐藏的this指针，不能访问任何非静态成员。**

它跟类的实例无关，只跟类有关，不需要this指针。

> ```c++
> class A
> {
> public:
> 	static void Func1()
> 	{
> 		_a = 20;
> 		_b = 10; //error：无法转换为this->_b
> 	}
> private:
> 	static int _a;
> 	int _b;
> };
> int _a = 10;
> ```



**【问题】**

- 静态成员函数可以调用非静态成员函数吗？

  >  不可以。因为非静态成员函数有第一个形参，且默认为this指针，而静态成员函数中是没有this指针的。即，静态成员函数不可调用非静态成员函数。
  >
  > ```c++
  > class A
  > {
  > public:
  > 	static void Func1()
  > 	{
  > 		Func2(); //error：没有this指针，无法准换为this->Fun2();
  > 	}
  >  
  > 	void Func2()
  > 	{
  > 		cout << "void Func2()" << endl;
  > 	}
  > };
  > int _a = 10;
  > ```



- 非静态成员函数可以调用静态成员函数吗？

  >   可以。因为静态成员函数和非静态成员函数都在类中，而在类中不受访问限定符的限制。
  >
  > ```c++
  > class A
  > {
  > public:
  > 	static void Func1()
  > 	{
  > 		cout << "static void Func1()" << endl;
  > 	}
  >  
  > 	void Func2()
  > 	{
  > 		Func1(); //正确：有this指针，可以转换为this->Func1;
  > 	}
  > };
  > int _a = 10;
  > ```