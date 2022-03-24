# csapp_lab

Lab Assignments for CSAPP

运行环境：

- Ubuntu 18.04 in Docker
- gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0

（Ubuntu 20.04 & Ubuntu in WSL2 不支持 32 位程序，[解决办法](https://stackoverflow.com/questions/42120938/exec-format-error-32-bit-executable-windows-subsystem-for-linux)）

## 1. Datalab

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

