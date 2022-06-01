### Phase 1

- 目标：学会打印
- 从 `read_six_numbers` 的第一个参数看出读入的是 `int` 整数，每个占 4 字节，而一个地址刚好指向一个字节。因此每读入一个数，PC 地址 +=4；该函数不会改变栈指针 `$rsp` 的值，即需要先分配空间。
- 进入 `scanf`  前的第二个参数 `rsi` 指明了输入的格式、类型；第一个参数 `rdi` 存的之前的输入；`scanf` 的参数从右到左依次进栈（因为传的地址，必须要在内存里面）；该函数的返回值 `rax` 是输入的个数。

### Phase 2

- 目标：学会查看函数参数
- 给一个内存地址赋值：`set {type}address = value`，如 `set {int}0x83040 = 4`，方便调试
- 以十六进制打印栈顶四十个字节：`x/40xb $rsp`

### Phase 3

- 目标：理解循环的汇编表示
- 含有循环，翻译 `cmp`、`test` 的时候，按照反过来的意思翻译更方便一点
- 里面有一段 `0x400f75 <phase_3+50>   jmpq   *0x402470(,%rax,8)` 间接跳转，可以使用 `x/8xb (0x402470+rax*8的值)` 打印内存地址，因为是小端表示，需要将输出反过来。如当 `rax=0` 时，跳转到 `0x0000000000400f7c`。后面根据输入的第一个数来决定输入的第二个数的意思，也就是有多组答案。如 `0 207`、`1 311 ` 等等

### Phase 4

- 目标：理解递归函数的汇编表示，或者学会调用函数
- 分析代码，要求第二个输入为 0，第一个输入作为 `func4` 的参数，函数返回值要求为 0
- 看不懂递归函数的时候，可以尝试直接用 `call` 暴力测试，随便测试了 0、1、3、7 都返回 0

下面是尝试翻译递归函数 `func4` 的过程（可能有误）：

```
记读入的第一个数为 a，第二个为 b
rsp -= 24
rcx = rsp+12 = b
rdx = rsp+8 = a
if (读入的个数 == 2) {
    if (a > 14) {
		bomb!
    }
    // <phase_4+39>   jbe    0x40103a <phase_4+46>
    edx = rdx = 14
    esi = rsi = 0
    edi = rdi = rsp+8 = a
    callq func4(a, 0, 14)
    if (eax == 0) {
    	if (b != 0) {
    		bomb!
    	}
    	// <phase_4+74>   je     0x40105d <phase_4+81>
    	rsp += 24
    	retq // 成功返回
    }
    // <phase_4+67>   jne    0x401058 <phase_4+76>
    bomb!
}
// <phase_4+32>   jne    0x401035 <phase_4+41>
bomb!

// 第一次简化
rax func4 (rdi, rsi, rdx) {
	rsp -= 8
	eax = edx 		// rax = rdx
	eax -= esi 		// rax -= rsi
	ecx = eax 		// rcx = rax
	ecx h >> 31 	// rcx = 其符号位 0 or 1
	eax += ecx 		// rax += rcx
	eax a>> 1 		// rax /= 2
	ecx = rax + rsi // rcx = rax + rsi
	if (ecx > edi) {
		edx = rcx-1 // rdx = rcx-1
		callq func4(rdi, rsi, rdx)
	}
	// <func4+22>     jle    0x400ff2 <func4+36>
	eax = 0 // rax = 0
	if (eax < edi)  {
		esi = rcx+1 // rsi = rcx+1
		callq func(rdi, rsi, rdx)
	}
	// <func4+43>     jge    0x401007 <func4+57>
	rsp += 8
    retq // 返回
}

// 第二次简化
rax func4 (rdi, rsi, rdx) {
	rax = rdx
	rax -= rsi
	rcx = rax
	rcx = (rcx<0)
	rax += rcx
	rax /= 2
	rcx = rax + rsi
	if (rcx > rdi) {
		rdx = rcx-1
		callq func4(rdi, rsi, rdx)
	}
	rax = 0
	if (rax < rdi)  {
		rsi = rcx+1
		callq func(rdi, rsi, rdx)
	}
    retq // 返回
}

// 第三次简化，尝试改写成 C 代码
int func4 (int rdi, int rsi, int rdx) {
	int rax = (rdx-rsi + (rdx-rsi < 0)) / 2;
	int rcx = (rdx-rsi + (rdx-rsi < 0)) / 2 + rsi;
	if (rcx > rdi) {
		rdx = rcx-1;
		rax = func4(rdi, rsi, rdx);
	}
	rax = 0;
	if (rax < rdi)  {
		rsi = rcx+1;
		rax = func4(rdi, rsi, rdx);
	}
    return rax;
}
```

### Phase 5

- 目标：通过偏移量获得答案
- 一个 ascii 字符是 8 位，可以用两个 16 进制表示，只需要最小的寄存器如 `cl`
- 若字符串存储在内存中：假设字符串为 "123456"，ascii 码为 31~36，末尾有一个`\0`，ascii 码为 0，起始地址存在 `rbx` 中，则 `($rbx+k)=30+k, k=1~6`，`($rbx+7)=0`。
- `man ascii` 可以查看对应表，compact table 使用方法：先看横表头，再看竖表头
- 二进制表示的 1~9 的末四位组成的数刚好等于该数

```
把输入的字符串存入 rbx
rax = fs:0x18 // 得到一个很大的数
(rsp+0x18) = rax
eax = 0		// rax = 0
do {
    ecx = (rbx + rax) = 一个字符	// rcx = 从第一个字符开始，每次取一个字符
    (rsp) = cl	// (rsp) = 该字符
    rdx = (rsp) = cl	// rdx = 第一个字符
	
	// 循环至此， rcx = (rsp) = rdx = 一个字符
    
    edx = edx & 0x00 00 00 0f // rdx = 字符末四位，结合下行，其实是一个十六进制的偏移量
    edx = (rdx + 0x4024b0) // 0x4024b0 开始存有字符串，计算偏移，从中取一个字符
    (16 + rsp + rax) = dl // 从 rsp+16 开始，存字符 = rdx
    rax += 1 // rax = 1
} while (rax == 6); // 重复直到遍历六个字符
(rsp+22) = (0) // 置为 ‘\0’
esi = (0x40245e) // esi = (0x40245e) = "flyers"
rdi = rsp+16 // rdi = 栈中保存的输入的字符串的首地址
比较 esi 和 rdi 是否相同，不同则 Bomb!
... 后面省略
```

### Phase 6

- 往没有运行过的地方 `jmp`（通常方向向下），反着翻译，翻译为 `if`
- 往运行过的地方 `jmp`（通常方向向上），正着翻译，翻译为 `while`
- 放弃了，近 300 行汇编代码，太长了，没时间做了

```
rsp -= 0x50 // 分配空间
r13 = rsp // r13 -> 第一个数
rsi = rsp // rsi -> 第一个数
读六个 int 整数，存到栈里
r14 = rsp // r14 -> 第一个数
r12d = 0x0 // r12 是一个控制循环次数的变量
.32:
rbp = r13
eax = (r13)
eax -= 1
if (eax > 5) bomb!
r12d += 1
if (r12d != 6) {
	ebx = r12d
	do {
        rax = ebx
        eax = (rsp + rax * 4)
        if ((rbp) == eax) bomb!
        ebx += 1
	} while (ebx <= 5);
	r13 += 4
	jmp 32 // 要跳出这个循环 r12d 要为 6
}
// 上面的嵌套循环测试六个数是否存在两个数相等，若存在则 bomb!
// <phase_6+60>   je     0x401153 <phase_6+95>
rsi = (rsp + 24)
rax = r14
ecx = 7
do {
    edx = ecx
    edx -= (rax)
    (rax) = edx
    rax += 4
} while (rax != rsi);
// 上面的循环让每个数 = 7-该数
esi = 0
.163:
ecx = (rsp + rsi)
if (ecx > 1) {
	eax = 1
	edx = ($0x6032d0) // 0x4c = 76
	// jmp 130
	do {
        rdx = (rdx + 8)
        eax += 1
	} while(eax != ecx);
	// jmp 148
	(0x20 + rsp + rsi*2) = rdx
	rsi += 0x4
	if (rsi != 0x18) {
		jmp 163
	}
	// jmp 183
	rbx = (rsp + 0x20)
	rax = rsp + 0x28
	rsi = rsp + 0x50
	rcx = rbx
	rdx = (rax)
	(rcx + 8) = rdx
	rax += 8
	
}
// <phase_6+169>  jle    0x401183 <phase_6+143>
```



