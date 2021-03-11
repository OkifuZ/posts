---
title: Everything (I know) about c++ const
date: 2021-03-04 19:01:05
tags:
excerpt: how const affect reference, pointer, function and class


---



const是c++中的修饰符，也是c++这门语言复杂性的一个原因。若没有充分理解const在多种情况下的含义，会成为代码可读性的重要敌人，也会造成众多难以察觉的错误。在这里我将我已知的const用法分情况进行了总结，用于日后遗忘时快速回顾。

## basic usage about const

const定义一种不可以改变值的变量，且定义时必须初始化。

```cpp
const int bufsize = 512; // 写法1
int const bufsize_ = 512; // 写法2 
```

编译过程中，对于上述赋值语句，编译器会在用到该变量的地方替换成对应的值。因此使用了const对象的文件都需要能够访问到初始值。为了实现这一功能，同时避免重复定义，编译器规定const对象只对当前文件内有效。

如果想多个文件共有const对象，则需要在生命与定义前加上extern关键字。



## reference，pointer and const

当const出现在引用与指针的定义或声明语句中时，有四种情况：

1. reference to const，对常量的引用

   定义方式为：

   ```cpp
   const int &capacity = bufsize; // 写法1
   const int &capacity = 512;
   int const &capacity_ = 512; // 写法2
   ```

   表示此引用所绑定的对象是不可变的，故不可以在定义之外对其赋值。

   ```cpp
   capacity = 1024 // error
   ```

   需要注意，与指针不同，没有所谓的`常量引用`，因为引用不是一种对象，而且对于一个普通引用，c++并没有改变其绑定对象的方法。

2. pointer to const，指向常量的指针

   ```cpp
   const int *capacity_pc = &bufsize; // 写法1
   int const *capacity_pc_ = &bufsize; // 写法2 
   ```

   `capacity_p`是指向常量对象bufsize的指针，这意味着可以通过普通的赋值语句改变`capacity_p`的指向，但是不可以通过解引运算符修改bufsize的值。

   ```cpp
   const int buffer_long = 1024;
   capacity_p = buffer_long; // ok
   *capacity_p = 1024; // error
   ```

3. const pointer，常量指针

   ```cpp
   int bufsize_longer = 2048;
   int *const capacity_cp = &bufsize_longer; 
   ```

   `bufsize_longer`是常量指针，因此无法使用赋值语句改变其指向的对象，但是可以使用解引运算改变其指向对象的值。

   ```cpp
   capacity_cp = &bufsize_longer; // error
   *capacity_cp = 4096;
   ```

4. const pointer to const，指向常量的常量指针 

   ```cpp
   const int *const capacity_cc = &bufsize; // 写法1
   int const *const capacity_cc = &bufsize; // 写法2
   ```



## initialization, copy and const

首先提出两个概念：对于const修饰的变量而言，如果其本身是常量（不可改变对象的值），则称其const修饰符为`顶层const`（high-level const）；如果该变量是指针或引用，且指向或绑定的对象是常量，则称其const修饰符为`底层const`（low-level const）。

如果仔细观察上述代码中含有const修饰的变量的赋值方式，结合编译器的规定，关于**初始化**，可以总结如下结论：

- 顶层const变量可以接受常量或非常量的赋值。
- 底层const变量只能接受同为底层变量的赋值。

同样的，对于**拷贝**，有如下结论：

- 顶层const变量可以拷贝赋值给常量或非常量的值。
- 底层const变量只能赋值给同为底层const的变量。



## function and const

对于**形参**的传递，其本质与变量的赋值无异，因此变量赋值的规则在这里同样适用。

1. 顶层const：

   在形参的初始化过程中，编译器会忽视掉顶层const修饰符，以保持和赋值语句性质的统一（顶层const变量可以同时接受常量或变量）。因此对于如下两个函数声明是一回事，同时出现时编译器会报错，而不是识别为重载函数

   ```cpp
   void foo(const int);
   void foo(int); // error
   
   ```

2. 底层const

   需要说明的只有常量引用（reference to const）作为形参的情形。

   如果函数`foo`的定义中形参是普通的引用，则受限于底层const赋值的机制，`foo`将无法传递**const对象**或**字面常量**给形参。这在有些时候会造成很多麻烦，如

   ```cpp
   bool is_pretty_name(string& s);
   bool is_pretty_name_(const string& s);
   
   ```

   在调用时

   ```cpp
   is_pretty_name("Kincaid"); // error, 字符串常量不可以传递给普通引用
   is_pretty_name_("Kincaid"); // absolutely ok
   
   ```

   这也说明了，编译器会区分仅仅是形参底层const不同的函数。

对于函数的返回值，其机制与变量赋值遵循同样的原理，这里不再赘述。

## class and const 

**常量成员函数**的是一类成员函数，其与一般成员函数的区别在于此函数的隐藏形参`*this`是**底层const修饰**的，如下函数`myFoo()`

```cpp
class MyClass {
  int number = 0;
  int myFoo();
  int myFoo_c() const; // 常量成员函数
};

```

通过参数列表之后的const，可以达到修饰`this`为底层const指针的效果。值得一提的是，`this`指针本身就是顶层const的，因为this天生与当前对象所绑定。

一个常量对象通常无法调用非常量成员函数，原因在于调用成员函数必须将一个指向当前对象的指针作为实参传递给隐藏的`*this`形参。如前所述，普通成员函数的`*this`形参是没有底层const修饰的，而指向常量对象的指针是底层const修饰的，因而后者无法赋值给前者。

因此，**一个常量对象（仅）可以调用常量成员函数**。

```cpp
const MyClass obj;
obj.myFoo(); // error
obj.myFoo_c(); // ok

```



此外，由于常量成员函数的`*this`指针底层const修饰，故在此函数内部**不可以更改对象的内容**。

```cpp
int MyClass::myFoo_c() {
    this->number = 1; // error
}

```



## type conversion and const

tbc...