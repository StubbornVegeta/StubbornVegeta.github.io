---
title: "C++程序的编译过程（一）"
date: 2022-02-25T13:43:00+08:00
image: "/categories/c++系列/cpp-blue-static.png"
categories:
    - C++系列
tags:
    - C++语法知识
---

C++源文件到可执行文件通常要经过预处理、编译、汇编和链接四个过程。
```
    *.cpp
      | <- 预处理
     *.ii
      | <- 编译
     *.s
      | <- 汇编
  *.o / *.obj
      | <- 链接
*.out / *.exe
```
## 预处理
预处理主要处理程序中的预处理指令，`#`开始的都叫做预处理指令，例如`#include`, `#define`等。如果你使用g++编译你的cpp文件，你可以通过添加`-E`参数来使你的编译器在预处理完之后，不进行后续操作，从而得到预处理后的文件。

假设在你当前目录下存在以下三个文件：
```bash
|-- main.cpp
|-- test.h
|-- test.cpp
```
三个文件的内容分别是：
```cpp
/***** main.cpp *****/
#include "test.h"

int main() {
    func();
    asm ( "movq $60, %rax \n"
          "movq $0,  %rdi \n"
          "syscall        \n");
}

/***** test.h *****/
void func();

/***** test.cpp *****/
void func() {

}
```
可以忽略`main.cpp`中的汇编代码，这只是为了保证手动链接不出错而添加的，可以不用理解这段代码。

在终端输入：
```bash
g++ -E main.cpp -o main.ii
```
`-o`表示输出结果到文件（`main.ii`）。上面这条指令执行完成之后，你将得到`main.ii`文件：

```cpp
# 0 "main.cpp"
# 0 "<built-in>"
# 0 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 0 "<command-line>" 2
# 1 "main.cpp"
# 1 "test.h" 1
void func();
# 2 "main.cpp" 2

int main() {
    func();
    asm ( "movq $60, %rax \n"
          "movq $0,  %rdi \n"
          "syscall        \n" );
}
```
`#include "test.h"`指令被替换成了`test.h`文件中的内容，复制头文件中的内容和宏展开是预处理环节主要做的事情，预处理环节没有语法检查功能。


## 编译
编译过程会将经过预处理之后的程序转换成特定汇编代码，通常可以使用`g++ -S`将源代码直接转换为汇编代码：
```bash
g++ -S main.cpp test.cpp
```
执行上述语句你将得到`main.s`和`test.s`两个文件：
```ASM
; ********** main.s **********
	.file	"main.cpp"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	call	_Z4funcv@PLT
#APP
# 5 "main.cpp" 1
	movq $60, %rax 
movq $0,  %rdi 
syscall        

# 0 "" 2
#NO_APP
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (GNU) 11.1.0"
	.section	.note.GNU-stack,"",@progbits


; ********** test.s **********
	.file	"test.cpp"
	.text
	.globl	_Z4funcv
	.type	_Z4funcv, @function
_Z4funcv:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	nop
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	_Z4funcv, .-_Z4funcv
	.ident	"GCC: (GNU) 11.1.0"
	.section	.note.GNU-stack,"",@progbits
```
g++输入参数中如果含有多个cpp文件，则分别对每一个cpp文件执行编译操作。

## 汇编
汇编会将汇编代码转换成机器码，也就是二进制的目标文件。你可以通过`g++ -c`来完成这步操作，也可以单独使用`as`指令来完成汇编过程。汇编过程会给每一个cpp文件产生一个目标文件。

```bash
g++ -c main.s test.s

# 或者使用as
as main.s -o main.o
as test.s -o test.o
```

## 链接
链接过程将多个目标文件以及所需的库文件链接成最终的可执行文件。这里你可以通过`g++`将多个目标文件链接起来，也可以使用`ld`进行链接。
```bash
g++ main.o test.o -o main
# 或者使用ld
ld -e main main.o test.o -o main
```
运行可执行文件，并通过echo查看进程返回值
```bash
$ ./main
$ echo $?
0
```
结果显示0，表示可执行文件链接成功。


