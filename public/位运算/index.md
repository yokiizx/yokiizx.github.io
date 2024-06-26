# 位运算


位运算，对二进制进行操作，有时候更加简练，高效，所以是很有必要学习和了解的。

> 基础的怎么变化的就不赘述了，工科生学 C 语言应该都学过吧，记住负数是以补码形式存在即可，补码 = 反码 + 1。

## 常用操作

-   `&` (有 0 为 0)： 判断奇偶性: `4 & 1 === 0 ? 偶数 : 奇数`
-   `|` (有 1 为 1)： 向 0 取整: `4.9 | 0 === 4`; `-4.9 | 0 === -4`
-   `~` (按位取反)： 加 1 取反: `!~-1 === true`,只有-1 加 1 取反后为真值;  
    `~~x`也可以取整或把 Boolean 转为 0/1
-   `^` ： 自己异或自己为 0: `A ^ B ^ A = B` 异或可以很方便的找出这个单个的数字（也叫无进位相加）
-   `x >> n` ：`x / 2^n`: 取中并取整 `x >> 1`
-   `x << n` ：`x * 2^n`
-   `x >>> n`: 无符号右移，有个骚操作是在 splice 时，`x >>> 0`获取要删除的索引可以用这个来避免对 -1 的判断，因为 `-1 >>> 0` 符号位的 1 会让右移 0 位后变成了一个超大的数字 42 亿多...

### 技巧

1. `n & (n-1)`：消除二进制中 n 的最后一位 1

    [191. 位 1 的个数](https://leetcode.cn/problems/number-of-1-bits/)

    ```js
    /**
     * @param {number} n - a positive integer
     * @return {number}
     */
    var hammingWeight = function (n) {
        let res = 0
        while (n) {
            n &= n - 1
            res++
        }
        return res
    }
    ```

    [231. 2 的幂](https://leetcode.cn/problems/power-of-two/)

    ```js
    /**
     * @param {number} n
     * @return {boolean}
     */
    var isPowerOfTwo = function (n) {
        if (n <= 0) return false
        // 2的n次方 它的二进制数一定只有一个1 去掉最后一个1应该为0
        return (n & (n - 1)) === 0
    }
    ```

2. `n & (~n + 1)`，提取出最右边的 1，这个式子即：n 和它的补码与。等价于 `n & -n`

3. `A ^ A === 0`：找出未成对的那个数字

    [268.丢失的数字](https://leetcode.cn/problems/missing-number/submissions/)

    ```js
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var missingNumber = function (nums) {
        const n = nums.length
        let res = 0
        for (let i = 0; i < n; ++i) {
            res ^= nums[i] ^ i
        }
        res ^= n
        return res
    }
    ```

## 补充：异或的运用

### 一组数，只有一个数出现了一次，其他都是偶数次，找出这个数

```java
int[] arr = new int[]{2, 1, 5, 9, 5, 4, 3, 6, 8, 12, 9, 6, 7, 3, 4, 2, 7, 1, 8};

/**
 * 找到唯一数字
 *
 * @param arr
 * @return
 */
public static int findOdd(int[] arr) {
    int res = 0;
    for (int num : arr) {
        res ^= num;
    }
    return res;
}
```

### 进阶：一组数，只有 2 个数出现了一次，其他都是偶数次，找出这 2 个数

lc.260

```java
/**
 *  根据上一题可以很容易得到 a^b 的值，问题在于怎么找到这两个数字
 * 1. 异或特性：相同为 0，不同为 1。所以可以找到 a^b 上的一位 1 -- rightOne 来区分 a 和 b
 * 2. rightOne 不断与每个数 &，结果不是 0 就是 rightOne 自身，也就可以把原来的数分为两组，相同的数一定进入一组
 * 3. 用另一个变量去与一组数再异或，就能剥离出一个不同的数，另一个数也就很容易得到了。
 *
 * @param arr
 * @return
 */
public static int[] findTwoOdd(int[] arr) {
    int var = 0; // 找到两个单独的数； a^b 的值
    for (int num : arr) {
        var ^= num;
    }

    // 找到最右边的 1  a&(~a + 1) 或者 a & -a
    int rightOne = var & -var

    int a = 0;
    for (int num : arr) {
        // num 与 rightOne 的结果只有两种 0 或者 rightOne 自身，根据这个区分开两组数
        if ((rightOne & num) == 0) {
            a ^= num;
        }
    }

    int b = a ^ var;

    return new int[]{a, b};
}
```

-   取最右边为 1 的数：`n & -n` 或者 `n & (~n + 1)`
-   消灭最右边的 1：`n & (n - 1)`，可以用来计算整数的二进制 1 的个数

## 其他位运算的技巧(待学习)

-   [Bit Twiddling Hacks](http://graphics.stanford.edu/~seander/bithacks.html#ReverseParallel)

