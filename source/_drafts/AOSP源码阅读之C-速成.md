---
title: AOSP源码阅读之C++速成
tags:
---
之前一直想着要先把 C++ 这玩意系统地梳理一遍再去阅读 AOSP 源码，因为里面涉及到 C++ 的部分到处都是，但是直接去学习一种程序设计语言实在是太枯燥了，并且 C++ 这玩意也是个庞然大物，想想还是放弃了，准备换个方式，在阅读 AOSP 源码的同时，不断地记录一些需要掌握的 C++ 语法和概念。

## C++基本语法
### [C 和 C++ 中的指针](https://liam.page/2017/02/05/pointer-in-C-and-Cpp/)
首先，指针也是变量，指针变量存放的是另外一个变量在内存中的地址。有趣的是，如果另外一个变量也是指针的话，这就有了多重指针的概念，看以下代码：

```c
int main(int argc, const char * argv[]) {
    int val  = 1024;
    int *p   = &val;
    int **pp = &p;
    std::cout << " val = " << val << endl;
    std::cout << "   p = " << p << endl;
    std::cout << "  *p = " << *p << endl;
    std::cout << "  pp = " << pp << endl;
    std::cout << " *pp = " << *pp << endl;
    std::cout << "**pp = " << **pp << endl;
    return 0;
}
```

这个程序的输出结果如下：

```bash
 val = 1024
   p = 0x7ffeefbff62c
  *p = 1024
  pp = 0x7ffeefbff620
 *pp = 0x7ffeefbff62c
**pp = 1024
Program ended with exit code: 0
```

分析程序之前，先理解一下这两个符号的作用：

* 地址运算符号 &amp;  
  
  获取一个变量在内存中的地址，地址表现形式为一个十六机制数。
  
* 间接寻址运算符 *
  
  指针变量存放的是一个地址，而 * 符号的作用就是找到这个地址在内存中的位置，然后把这个位置储存的值取出来。
  
main()函数总共声明了 3 个变量，分别是 val、p、pp，其中 val 是 int 类型的变量，p、pp 是 int 类型的指针变量。如果理解了 & 符号的作用，这三个变量的值应该很好理解：

| 变量 | 值 | 
| --- | --- |
| val | 就是一个 int 值，等于1024 | 
| p | val 变量的地址，一个十六进制数 | 
| pp | p 变量的地址，一个十六进制数 | 

如果理解了 * 符号的作用，那么理解下面这些东西的值也不是难事：

| 变量 | 值 | 
| --- | --- |
| \*p | 因为 p 变量存放的是 val 变量的地址，找到这个地址，取出它的值，即 val 变量的值，所以 *p = val = 1024 | 
| \*pp | 因为 pp 变量存放的是 p 变量的地址，找到这个地址，取出它的值，即 p 变量的值，所以 \*pp = p = &val，也就是 \*pp 等于 val 变量的地址 | 
| \**pp | 因为 \*pp = p，所以 \*\*pp = \*p = val = 1024，也就是 \*\*pp 等于 1024 | 
</br>
## 概念
### sp、wp强弱指针
跟Java不同，C++本身并不具备内存回收机制，Android 提供了 sp、wp 这两种智能指针，在普通指针上加了一层封装，通过引用计数法来自动管理内存的回收。（暂时知道sp、wp是啥，了解大概原理就行了，有时间再详细地阅读以下的文章）
* [Android系统的智能指针（轻量级指针、强指针和弱指针）的实现原理分析](https://blog.csdn.net/Luoshengyang/article/details/6786239)
* [如何理解智能指针？](https://www.zhihu.com/question/20368881)
* [Binder学习笔记（十一）—— 智能指针](http://palanceli.com/blog/2016/06/10/2016/0610BinderLearning11/)








