---
layout: post
title: C语言中的存储类型
category: programming
tags: [C language, linux]
---
今天去面试，意外地被问到了一些C语言基础的问题，如结构体字节对齐、static修饰符修饰函数和变量的用法、结构体数组的看代码写结果题。基本上都答出来了，不过static的那个稍微有点模糊。翻阅了一下C语言和操作系统的相关书籍，略做整理如下。

在C语言中，每一个函数和变量都有**数据类型**和**存储类型**，存储类型的 4 个关键字分别是 auto（自动），extern（外部），static（静态），register（寄存器）。

其中变量的存储类型决定了变量的作用域、存储分配方式、生命周期和初始化方式。而函数的存储类型决定了函数的作用域。先说变量：

<!-- excerpt -->

1. auto
可缺省不写，局部变量的缺省存储类型为 auto 。作用域是变量所处的块，典型的就是一个函数内，另外也可以用一对花括号来定义一个块。在栈上分配内存。退出块时释放内存，其所存的值也随之消失。没有缺省初始值。若有显式的初始化赋值，每次进入该块都会重新执行。

2. extern 外部变量存储类型
定义一个外部变量时是不需要 extern 关键字的，只需把变量定义在 C 文件中函数之外即可。在其他 C 文件中用 extern 关键字来声明一个外部变量时，相当于将该变量的作用域扩展到了该C文件中。如果只是在其他文件的某一个函数中声明，则该外部变量在此文件中的作用域只有那一个函数。外部变量的缺省初始值为0。
同名的局部变量会在函数内屏蔽外部变量。

3. static
该修饰符可以修饰外部或局部变量。静态变量的生命周期为整个程序的生命周期，缺省初始值为 0 。
静态局部变量。所在块退出时，值不会变。若显式赋初值，只在第一次进入该块时执行。
静态外部变量。和普通外部变量几乎等同，只是将该变量的作用域限定在了本文件中。

4. register
register int i;
只能作用于局部变量，只代表向编译器建议使用高速硬件寄存器来存储该变量，是否实施取决于编译器的支持和实现（和 inline 显式定义内联函数类似），若不支持，则当作自动变量处理。可用于访问最频繁的变量。
寄存器变量不能取地址运算。函数形参可以声明为 register 类型，但是不能声明为 extern 或 static 。

函数：
函数的存储类型只有extern和static两种。缺省存储类型为extern，static将该函数的作用域限定在本文件，可以用来控制最小的访问权限，也可以解决在不同文件中使用了同名函数的问题。

在 Linux 的 ELF 可重定位目标文件中，初始化显式赋为非 0 值的静态变量和外部变量存于 .data 节中，而缺省初始值为 0 或者显式赋初值为 0 的静态变量和外部变量存于 .bss节中，由于已经确定了这些变量的初始值，在目标文件中 .bss 节不占据实际的空间，它仅仅是一个占位符，从而提高了空间效率。这也是区分这些变量是否初始化的意义。最终加载可执行的目标文件到运行时的虚拟存储器映像中时，.data 和 .bss节都作为了读/写存储段加载，自动变量在用户栈上分配存储空间。