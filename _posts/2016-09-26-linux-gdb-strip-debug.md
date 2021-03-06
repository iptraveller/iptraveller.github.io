---
layout: post
title: 调试不带符号表的coredump文件
excerpt: ""
categories: [linux]
comments: true
---

## 背景介绍
在小内存的嵌入式设备中，为了压缩进程的大小，通常是去掉符号表后在设备上运行，此时进程崩溃后生成的core文件和对应的二进制文件是不带符号表的，使用gdb进行查看时无法获取到准确信息。

但是我们可以单独保存被删掉的符号表，在gdb的时候进行导入来进行调试。

## 例子
以一个二进制进程链接一个动态库为例<br>
动态链接库为命名为b.c
{% highlight c %}
#include <stdio.h>

int b(int (*a)(void))
{
	return a() + 100;
}
{% endhighlight %}
主程序命名为a.c
{% highlight c %}
#include <stdio.h>

extern int b(int (*a)(void));

int a() 
{
	int *a = NULL;
	*a=0;
	return 1;
}

int c()
{
	return b(a) + 1000;
}

int main()
{
	c();
	return 0;
}
{% endhighlight %}

1. 生成动态链接库及主程序  
    **gcc -g  b.c -fPIC -shared -o libb.so**  
    **gcc -g a.c -o a.out -L. -lb**
2. 提取主程序及动态链接库的符号表  
    **strip --only-keep-debug -o a.sym a.out**  
    **strip --only-keep-debug -o libb.sym libb.so**
3. 删除主程序及动态链接库的符号表  
    **strip a.out**  
    **strip libb.so**
4. 运行程序产生core文件
{% highlight bash %}
    root@oa:/home/share/gdb# ./a.out
    Segmentation fault (core dumped)
{% endhighlight %}

默认未导入符号表的情况下，进行gdb，看到一堆的??和No symbol
{% highlight bash %}
root@oa:/home/share/gdb# gdb a.out core-a.out-4156
Core was generated by `./a.out'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x0804854d in ?? ()
(gdb) bt fu
#0  0x0804854d in ?? ()
No symbol table info available.
#1  0xb775a4f6 in b () from /home/share/gdb/libb.so
No symbol table info available.
#2  0x0804856c in ?? ()
No symbol table info available.
#3  0x0804857e in ?? ()
No symbol table info available.
#4  0xb75afa83 in __libc_start_main (main=0x8048573, argc=1, argv=0xbfd86fd4, init=0x8048590, fini=0x8048600, rtld_fini=0xb776f180 <_dl_fini>, stack_end=0xbfd86fcc) at libc-start.c:287
#5  0x08048461 in ?? ()
No symbol table info available.
{% endhighlight %}

这时候使用***-symbols***参数的gdb命令，导入主程序符号表并启动gdb
{% highlight bash %}
root@oa:/home/share/gdb# gdb --exec=a.out --core=core-a.out-4156 -symbols=a.sym
Core was generated by `./a.out'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x0804854d in a () at a.c:8
8	 *a=0;
(gdb) bt  fu
#0  0x0804854d in a () at a.c:8
        a = <error reading variable a (can't compute CFA for this frame)>
#1  0xb775a4f6 in b () from /home/share/gdb/libb.so
No symbol table info available.
#2  0x0804856c in c () at a.c:14
No locals.
#3  0x0804857e in main () at a.c:19
No locals.
{% endhighlight %}

此时发现主程序的函数有符号表了，但是动态库还只有函数，没有变量符号，还是提示No symbol。

导入动态库的符号表，需要先使用***info shared***命令找到动态库链接的位置：
{% highlight bash %}
(gdb) info shared
From        To          Syms Read   Shared Object Library
0xb775a3c0  0xb775a4fb  Yes (*)     /home/share/gdb/libb.so
0xb75ad420  0xb76df68e  Yes         /lib/i386-linux-gnu/libc.so.6
0xb7760860  0xb77787ac  Yes         /lib/ld-linux.so.2
(*): Shared library is missing debugging information.
{% endhighlight %}

然后使用***add-symbol-file***命令将动态库的符号表导入，此时所有的符号表都有了。
{% highlight bash %}
(gdb) add-symbol-file libb.sym 0xb775a3c0
add symbol table from file "libb.sym" at
    .text_addr = 0xb775a3c0
(y or n) y
Reading symbols from libb.sym...done.
(gdb) bt fu
#0  0x0804854d in a () at a.c:8
        a = <error reading variable a (can't compute CFA for this frame)>
#1  0xb775a4f6 in b (a=0x804853d <a>) at b.c:5
No locals.
#2  0x0804856c in c () at a.c:14
No locals.
#3  0x0804857e in main () at a.c:19
No locals.
{% endhighlight %}

如果提示找不到动态库的链接路径，则可以使用***set solib-search-path \<path\>***命令来设置动态库的链接路径。

## 相关命令
* ***strip --only-keep-debug -o \<bin.sym\> \<bin\>***
* ***strip \<bin\>***
* ***gdb --exec=\<bin\> --core=\<core\> -symbols=\<bin.sym\>***
* ***info shared***
* ***add-symbol-file \<lib.sym\> \<address\>***
* ***set solib-search-path \<path\>***
* ***show solib-search-path***
