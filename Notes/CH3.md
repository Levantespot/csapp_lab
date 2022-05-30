## CH3

### Notes

- 基址和变址寄存器都必须是64位寄存器。

- MOV 指令的源和目的类型中，不包括内存到内存。
  - 立即数 - 寄存器
  - 立即数 - 内存
  - 寄存器 - 寄存器
  - 寄存器 - 内存
  - 内存 - 寄存器

- MOV 指令的指令后缀，对于「寄存器 - 寄存器」，要匹配 `max(源,目的)`，其余四种只需要匹配 `min(源,目的)`。

- 栈底 -> 栈顶，地址以 8 为单位减少。即栈顶地址最低，栈底最高。
  - 压栈 `pushhq %rbp` 等价于 `PC=PC-8`，即 `subq $8, %rsp` + `movq %rbq, (%rsp)`
  
  - 出栈 `popq %rbp` 等价于 `PC=PC+8`，即 `movq (%rsp), %rax` +  `addq $8, %rsp`

  - 程序可以通过常用的地址偏移来访问栈内非栈顶元素。
  
  - 访问栈不一定必须通过压栈和出栈，可以直接对 `$rsp` 的值进行运算，模拟出入栈，此时 PC 的值不一定以 8 为单位改变，要看实际情况。如用 `scanf` 读取 2 个 `int` 整数（四字节），可以是如下情况：
  
    ```
    // 以下为方便描述，简化、改写一些步骤
    sub 0x8, $rsp // rsp -= 8，即分配 2*4=8 个字节
    move $0x2, $(rsp+4) // 栈顶+4偏移读入第二个 int 整数
    move $0x1, $rsp // 栈顶读入第一个 int 整数
    ```
  
    如果读入一个字符串，就可能是每读一个字符 PC+1。
  
- `leq` vs `mov`:  [ref](https://stackoverflow.com/questions/46597055/using-lea-on-values-that-arent-addresses-pointers)

  - 语法基本相同，但对于变址寻址和间接地址，`mov` 会访存，而 `leq` 不会；并且 `leq` 的目标必须是一个寄存器。

  - `mov`

    ```
    1 movl $0x4050,%eax 	Immediate--Register, 4 bytes
    2 movw %bp,%sp 			Register--Register, 2 bytes
    3 movb (%rdi,%rcx),%al 	Memory--Register, 1 byte
    4 movb $-17,(%esp) 		Immediate--Memory, 1 byte
    5 movq %rax,-12(%rbp) 	Register--Memory, 8 bytes
    ```

  - `leq` 

    - 类似 `mov`，计算地址，然后搬运：`leq %rsp, %rax` 和 `move $rsp, $rax` 等价，都是访问寄存器
    - `leq 8(%rsp), %rax` 是寄存器到寄存器，`move 8(%rsp), %rax` 是内存到寄存器
    - 可以简洁的描述算数运算，所以有时它的使用与有效地址的计算无关，如：`leq 17(%rax, %rax, 3), %rax`，将 `4*rax+17` 存入 `rax`

- 当执行 PC 相对寻址时，PC 的值是跳转指令 `jmp` 后面指令的地址。P140

  ```
  // assembly code
  movq %rdi, %rax
  jmp .L2
  .L3:
  sarq %rax
  .L2:
  testq %rax, %rax
  jg .L3
  8 rep; ret
  
  // disassembled code
  0: 48 89 f8 	mov %rdi,%rax
  3: eb 03 		jmp 8 <loop+0x8>	// 0x8 = 0x03 + 0x5(下面一行的地址)
  5: 48 d1 f8 	sar %rax
  8: 48 85 c0 	test %rax,%rax
  b: 7f f8 		jg 5 <loop+0x5>		// 0x8 = 0xf8(补码) + 0xd(下面一行的地址)
  d: f3 c3 		repz retq
  ```

- 条件分支的实现方法

  - 条件转移 `cmp` + `jmp`：根据条件，跳转到对应分支
  - 条件传送 `cmp` + `cmov`：计算两个分支的结果 A 和 B，先把 A 存到 `rax` 中，若不满足条件，则把 `B` 存到 `rax`，最后返回 `rax`。性能更高，但是应用场景受限。

- 转移控制：控制从函数 P 转移到函数 Q：

  1. 在函数 P 中，使用 `call addr(Q)`

  2. 将程序计数器 PC 置为 Q 的起始地址：`%rip = addr(Q)`

  3. 如果参数（类型为整数和指针）数量大于 6，需要把额外的参数压入栈

     - 后面的参数先进栈，参数 7 位于栈顶
     - 访问栈中的参数时使用「栈顶地址 + 偏移量」的地址访问，如访问参数 7：`8(%rsp)`

  4. 将 P 中调用 Q 的下一条语句的地址（返回地址）压入栈：`%rsp-=8` + `(%rsp)=next_addr(Q)`

  5. 如果需要将局部变量入栈：

     1. 先为局部变量分配内存空间（栈帧）：`%rsp-=16` 代表分配 16 字节
     2. 使用偏移地址访问栈中局部变量：`8(%rsp)` 代表访问第一个局部变量
     3. 使用完局部变量后，撤销分配的内存空间：`%rsp+=16` 代表撤销 16 字节
     4. 撤销局部变量的栈帧后，`%rsp` 应该指向返回地址

     局部变量也需要存入栈中的情况：

     - 寄存器不足够存放所有的本地数据
     - 对一个局部变量使用地址运算符 `&`，因此需要存入内存，以获得地址（寄存器无地址）
     - 某些局部变量是数组或结构

  6. Q 中运行到 `ret` 时，出栈，得到 Q 的下一条语句的地址，并赋值给 PC：`%rsp+=8` + `%rip=next_addr(Q)`

     - 出栈时，额外参数数量是知道的，应该也会同时将这些参数的栈帧释放

- 被调用者保存 (callee-saved) 寄存器和调用者保存 (caller-saved) 寄存器，代表保存这些寄存器的负责人：调用者负责保存的，被调用者就可以随意使用；被调用者负责保存的，则返回调用者时应保持和被调用前一致。

  - 寄存器 `%rbx`、`%rbp` 和 `%r12`～`%r15` 被划分为被调用者保存寄存器。

- 数据对齐：对齐原则是任何 K 字节的基本对象的地址必须是 K 的倍数。如 `long` 类型的变量的起始地址必须是 8 的倍数，不够就插入间隙。

- pass



### 习题

3.18

```c
#rdi	rsi		rdx
#x		y		z
long test(long x, long y, long z){
    long val = x+y; // leaq
    val = val + z;	// addq
    if (x + 3 < 0) { // cmpq -3, rdi
        if (y - z < 0) // cmpq rdx, rsi
            val = x * y; // movq rdi rax + imulq rsi rax
        else // jge .L3
            val = y * z; // movq rsi rax + imulq rdx rax   
    } else { // jge .L2
        if (x - 2 > 0) // cmpq 2 rdi
            val = x * z;
        else // jle .L4
            return val;
    }
}

long test(long x, long y, long z){
    long val = x+y+z; // leaq + addq
    if (x < -3) { // cmpq -3, rdi
        if (y < z) // cmpq rdx, rsi
            val = x * y; // movq rdi rax + imulq rsi rax
        else // jge .L3
            val = y * z; // movq rsi rax + imulq rdx rax   
    } else if (x > 2) // jge .L2 + cmpq 2 rdi
        val = x * z;
    return val; // jle .L4
}
```

3.20

```c
rax = x+7;
if (x > 0) rax = x; // testq rdi rdi + cmovns
rax = rax >> 3;
return rax;
// equal to:
return x / 8;
```

3.34

stack:

```
	1	2	3	4	5	6	7	8
-----------------------------------
								r15
								r14
								r13
								r12
								rbp
								rbx (24)
								.. (16)
								.. (8)
								.. (0) <-rsp

```

3.38

```
2	rdx = 8 * rdi				// rdx = 8*i
3	rdx = rdx - rdi				// rdx = 7*i
4	rdx = rdx + rsi				// rdx = 7*i + j
5	rax = 5 * rsi				// rax = 5*j
6	rdi = rdi + rax				// rdi = i + 5*j
7	8 * (j * M + i) = 8 * rdi	// j * M + i = j*5 + i 
8	8 * (i * N + j) = 8 * rdx	// i * N + j = i*7 + j

-> M = 5, N = 7
```









