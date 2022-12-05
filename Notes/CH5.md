# CH5 优化程序性能

### 1. 使用编译器的优化选项，

如：在 `gcc` 的参数中指定优化级别 `-O1`、`-O2`。

### 2. 消除循环的低效率

**代码移动**：将需要执行多次（如循环中）但是计算结果不变的计算移动到不会被多次求值的部分。

应用 1：

```c
// 优化前
for(int i = 0; i < size(array); i++)
    ...
// 优化后
int length = size(array);
for( int i = 0; i < length; i++)
    ...
```

### 3. 减少过程调用

### 4. 消除不必要的内存引用

应用 1：

```c
// 优化前
int sum(int *array, int length, int *dest) {
    *dest = 0;
    for (int i = 0; i < length; i++) {
        *dest += array[i];
    }
}
// 优化后
int sum(int *array, int length, int *dest) {
    int sum = 0;
    for (int i = 0; i < length; i++) {
        sum += array[i];
    }
    *dest = sum;
}
```

解释：通过检查编译代码发现，修改前的汇编代码会有重复的内存引用。

### 5. 循环展开

通过增加每次迭代计算的元素的数量，减少循环的迭代次数。

- 可以减少循环索引计算和条件分支

- 有利于流水线并发执行

应用 1：

```c
// 优化前
int sum(int *array, int length, int *dest) {
    int sum = 0;
    for (int i = 0; i < length; i++) {
        sum += array[i];
    }
    *dest = sum;
}
// 优化后（以展开次数 2 优化）
int sum(int *array, int length, int *dest) {
    int sum = 0;
    int limit = length-1;
    // 每次迭代结合 2 次计算
    for (int i = 0; i < limit; i+=2) {
        sum = (sum + array[i]) + array[i+1];
    }
    // 处理剩下的计算
    for ( ; i < length; i++) {
        sum += array[i];
    }
    *dest = sum;
}
```

### 6. 并行计算
