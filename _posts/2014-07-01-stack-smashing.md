---
layout: post
title: 越界写内存造成的程序崩溃问题
category: programing
tags: [linux, gdb]
---

最近在 linux 下做一个无线防护系统的配置管理模块，测试运行一部分代码时程序崩溃，出现了下方的提示：

{:.terminal}
    *** stack smashing detected ***: /home/work/wlanconfd/wlancfd terminated
    ======= Backtrace: =========
    /lib/i386-linux-gnu/libc.so.6(__fortify_fail+0x45)[0xb7f23eb5]
    /lib/i386-linux-gnu/libc.so.6(+0x104e6a)[0xb7f23e6a]
    /home/work/wlanconfd/wlancfd[0x804906a]
    /home/work/wlanconfd/wlancfd[0x8049368]
    /home/work/wlanconfd/wlancfd[0x804861f]
    /lib/i386-linux-gnu/libc.so.6(__libc_start_main+0xf3)[0xb7e384d3]
    /home/work/wlanconfd/wlancfd[0x8048581]
    ======= Memory map: ========
    08048000-0804a000 r-xp 00000000 08:01 1180859    /home/work/wlanconfd/wlancfd
    0804a000-0804b000 r--p 00001000 08:01 1180859    /home/work/wlanconfd/wlancfd
    0804b000-0804c000 rw-p 00002000 08:01 1180859    /home/work/wlanconfd/wlancfd
    0804c000-0806d000 rw-p 00000000 00:00 0          [heap]
    b7dee000-b7e0a000 r-xp 00000000 08:01 1049431    /lib/i386-linux-gnu/libgcc_s.so.1
    b7e0a000-b7e0b000 r--p 0001b000 08:01 1049431    /lib/i386-linux-gnu/libgcc_s.so.1
    b7e0b000-b7e0c000 rw-p 0001c000 08:01 1049431    /lib/i386-linux-gnu/libgcc_s.so.1
    b7e1e000-b7e1f000 rw-p 00000000 00:00 0 
    b7e1f000-b7fc3000 r-xp 00000000 08:01 1053268    /lib/i386-linux-gnu/libc-2.15.so
    b7fc3000-b7fc5000 r--p 001a4000 08:01 1053268    /lib/i386-linux-gnu/libc-2.15.so
    b7fc5000-b7fc6000 rw-p 001a6000 08:01 1053268    /lib/i386-linux-gnu/libc-2.15.so
    b7fc6000-b7fc9000 rw-p 00000000 00:00 0 
    b7fd9000-b7fdd000 rw-p 00000000 00:00 0 
    b7fdd000-b7fde000 r-xp 00000000 00:00 0          [vdso]
    b7fde000-b7ffe000 r-xp 00000000 08:01 1052972    /lib/i386-linux-gnu/ld-2.15.so
    b7ffe000-b7fff000 r--p 0001f000 08:01 1052972    /lib/i386-linux-gnu/ld-2.15.so
    b7fff000-b8000000 rw-p 00020000 08:01 1052972    /lib/i386-linux-gnu/ld-2.15.so
    bffdf000-c0000000 rw-p 00000000 00:00 0          [stack]
    
    Program received signal SIGABRT, Aborted.

<!-- excerpt -->

应该是程序中发生了栈溢出，导致了函数的返回指针被修改，进而函数返回时会发现返回的代码段指针错误，提示上述的"stack smashing detected"。用 gdb 调试，在出现上述崩溃信息后用 bt命令来打印当前的函数调用栈的全部信息如下：

{:.terminal}
    #0  0xb7fdd424 in __kernel_vsyscall ()
    #1  0xb7e4d1df in raise () from /lib/i386-linux-gnu/libc.so.6
    #2  0xb7e50825 in abort () from /lib/i386-linux-gnu/libc.so.6
    #3  0xb7e8a39a in ?? () from /lib/i386-linux-gnu/libc.so.6
    #4  0xb7f23eb5 in __fortify_fail () from /lib/i386-linux-gnu/libc.so.6
    #5  0xb7f23e6a in __stack_chk_fail () from /lib/i386-linux-gnu/libc.so.6
    #6  0x08048fc4 in init_sta_obj_cfg () at imp_cfd.c:177
    #7  0x0804929c in init_wlan_cfg () at imp_cfd.c:218
    #8  0x080485ef in main (argc=1, argv=0xbffff294) at wlan_cfd.c:8

问题出在 init_sta_obj_cfg() 函数返回时。仔细检查了下函数内部的代码，发现了问题应该出在一个将 mac 地址从字符串形式转换到内存形式的内联函数上。

{% assign lang = 'c' %}
{% capture codeblock %}static inline void str2mac(const char *mac_str, __u8 mac[])
{
    sscanf(mac_str, "%x:%x:%x:%x:%x:%x",
        &mac[0], &mac[1], &mac[2], &mac[3], &mac[4], &mac[5]);
}
{% endcapture %}
{% include AH/print_code %}/

这里 __u8 的定义是 `typedef unsigned char __u8;` 。在编译时其实就已经给出了警告：

{:.terminal}
    警告： 格式 ‘%x’ expects argument of type ‘unsigned int *’, but argument 3 has type ‘__u8 *’ [-Wformat]

我之前误以为会将 '%x' 读取到的16进制数隐式转换成 __u8 类型存入到变量中，可事实并非如此。做了个小实验如下：

{% assign lang = 'c' %}
{% capture codeblock %}#include<stdio.h>

int main()
{
    unsigned char x;

    scanf("%x", &x);
    printf("%x\n", x);

    return 0;
}
{% endcapture %}
{% include AH/print_code %}/
用 gdb 调试来单步执行，从标准输入读取 x 之前，用 p/x 命令查看 x 及该地址之后的四个字节处的16进制值如下：

{:.terminal}
    (gdb) p/x x
	$1 = 0xb7
    (gdb) p/x *(&x)
    $2 = 0xb7
    (gdb) p/x *(&x+1)
    $3 = 0xf0
    (gdb) p/x *(&x+2)
    $4 = 0x84
    (gdb) p/x *(&x+3)
    $5 = 0x4
    (gdb) p/x *(&x+4)
    $6 = 0x8

输入字符 "dd" 之后再次打印查看：

{:.terminal}
    (gdb) n
    dd
    8         printf("%x\n", x);
    (gdb) 
    dd
    10        return 0;
    (gdb) p/x x
    $7 = 0xdd
    (gdb) p/x *(&x)
    $8 = 0xdd
    (gdb) p/x *(&x+1)
    $9 = 0x0
    (gdb) p/x *(&x+2)
    $10 = 0x0
    (gdb) p/x *(&x+3)
    $11 = 0x0
    (gdb) p/x *(&x+4)
    $12 = 0x8

由上可以很清楚的看出，虽然输入的16进制数 "dd" 中只占1字节，但 '%x' 的确是按照 ‘unsigned int *’ 类型来读取的，在32位的小端系统上，将 x 地址增大方向上的3个字节都置为了0 。这样会造成未知的内存区域被修改，从而产生安全隐患。再看上面的那个内联函数，其实每次将 '%x' 读取的值写入 mac[i] 处时都会越界写后面地址的内存区域，只不过每次又被新写入 mac[i+1] 的正确值覆盖。然而，当写到 mac[5] 时，就会越界写后面未知的内存区域了。上面的程序崩溃应该就是越界写修改了函数的返回指针，进而函数返回时发现返回的代码段指针错误造成的。

将该内联函数修改为如下形式后问题得到了解决。

{% assign lang = 'c' %}
{% capture codeblock %}static inline void str2mac(const char *mac_str, __u8 mac[])
{
    unsigned int temp[6];
    int i;

    sscanf(mac_str, "%x:%x:%x:%x:%x:%x",
        &temp[0], &temp[1], &temp[2], &temp[3], &temp[4], &temp[5]);
    for (i = 0; i < 6; i++) {
        mac[i] = (__u8)temp[i];
    }
}
{% endcapture %}
{% include AH/print_code %}

先将6个值存入 unsigned int 类型的数组中，再强制转换类型一一存入 __u8 类型的数组中。