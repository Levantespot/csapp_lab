# csapp_lab

Lab Assignments for CSAPP

运行环境：

- Linux version 5.10.102.1-microsoft-standard-WSL2
- gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0

## 1. Datalab

- 对应章节：第二章
- 需要的知识
  - 整数、浮点数在计算机中如何表示、存储、运算
  - 按位运算符 `!^~&`、逻辑运算符 `&& ||`

可能需要的前置：

```shell
$ sudo apt-get install build-essential # libc6-dev、libc-dev、gcc、g++、make、dpkg
$ sudo apt-get install gcc-multilib # lib
```

运行：

```shell
# 进入实验目录
$ cd datalab-handout

# 检查代码是否符合要求（是否通过编译、是否满足最大运算次数等）。若符合要求，则什么都不会显示。
$ ./dlc bits.c 

# 编译
$ make
# 检查代码结果是否正确，并打分
$ ./btest 

# 自己写的脚本，包含上面三行
$ ./my_test.sh
```

## 2. Bomb

- 对应章节：第三章
  - [笔记](Notes/CH3.md)

- 需要的知识：
  - 汇编基础知识：16 个寄存器的用处，指令的运算、操作、移动
  - 使用 gdb 阅读汇编代码，能够反汇编出原来的 C 语言代码。
    - [gdb 笔记](https://github.com/Levantespot/Cheatsheets/blob/main/GCC%26G%2B%2B%26GDB.md)
- [解决过程](Notes/bomb.md)（第六题没做完）

```
$ gdb bomb
(gbd) b 74		// phase 1
(gbd) b 82		// phase 2
(gbd) b 89		// phase 3
(gbd) b 95		// phase 4
(gbd) b 101		// phase 5
(gbd) b 108		// phase 6
r < ans.txt	// ans.txt 存放答案，每行一个 phase 的答案
```





