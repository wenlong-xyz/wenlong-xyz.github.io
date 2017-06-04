---
title: Little Tips（持续更新）
date: 2016-05-25 20:31:37
category: Technology
toc: true
---
&emsp;&emsp;收录一些开发中的小技巧
# Code Style
## Java 
* 源文件
  - 编码：UTF-8
  - 文件结构（各部分之间有且仅有一个空行）：
    + License or copyright 信息
    + Package 声明
    + Import 声明：无通配符import
    + 一个top-level类
* 大括号
  - 非空代码块
    + 左括号前无换行，后边换行
    + 右括号前有换行，后边也有换行（if后紧跟的else除外，catch等除外）
    ```java
    
      return new MyClass() {
        @Override public void method() {
          if (condition()) {
            try {
              something();
            } catch (ProblemException e) {
              recover();
            }
          }
        }
      };
    ```
  - 空代码块
    + 左右括号紧跟或者换一行
    ```java
      // This is acceptable
      void doNothing() {}

      // This is equally acceptable
      void doNothingElse() {
      }
    ```
* 缩进
  - 两个空格（目前习惯缩进四个空格）
* 自动换行
  - 类限制，一行不能过长，限制为80或100字符
  - 从非赋值运算符出断开，在符号前断开（比如+，它将位于下一行）
  - 赋值运算符后断开
  - 方法名或者构造函数名与左括号留在同一行
  - 都好与前面的内容留在同一行
  - 自动换行时，第一行后的每一行至少比第一行多缩进4个空格
* 空行
  - 类内连续的成员之间：字段（可选）、构造函数、方法等
  - 函数体中，语句的逻辑分组间使用空行
* 水平空格
  - 保留字与紧随其后的`(`之间（如if, for等）
  - 保留字与其前面的`}`
  - `{`前，例外：
    + `@SomeAnnotation({a, b})`
    + `String[][] x = {{"foo"}};`
  - 二元或三元运算符两侧
  - `,` `:` `;`及`)`后
  - 注释`//`后
  - 数组初始化时，大括号内的空格是可选的`new int[] {5, 6}` 和 `new int[] { 5, 6 }` 都是可以的
* 其他
  - 变量声明：一次只声明一个变量，需要时才声明，并尽量进行初始化

Reference：
1. [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)



# 小技巧
```java
int mid = left + ((right - left) >> 1)   // 二分法,注意便面的括号
int mid = (left + right) / 2           // ！！！不要使用，因为如果left或right很大，二者的和可能溢出，但mid不会溢出

/**************************** 位运算 ****************************/
(1 << 31) - 1; // 获取int型最大值，由于优先级关机，括号不可以省略
1 << 31;       // 获取int型最小值
(n & 1) == 1;  // 判断一个数的奇偶性
(x ^ y) >= 0   // 判断x,y正负符号是否相同
n > 0 ? (n & (n - 1)) == 0 : false;                     //判断n是不是2的幂
n > 0 && (1162261467 % n) == 0;     //1162261467 = 2 ^13//判断n是不是3的幂 
n > 0 && (n & (n - 1)) == 0 && (n & 0x55555555) != 0;   //判断n是不是4的幂
m & (n - 1);           // <==> m % (n - 1) 其中 n = 2 ^ k, k位正整数
(n >> (m - 1)) & 1;    // 从低位到高位，取n的第m位
n & (1 << (m - 1)) == 0// 从低位到高位，取n的第m位 
n | (1 << (m - 1));    // 从低位到高位，将n的第m位置1
n & ~(1 << (m - 1));   // 从低位到高位，将n的第m位置0
Integer.bitCount(num); // 统计每位1的个数       
((1 << i) & num) == 0  // 判断某一位是否为0
num = ((1 << i) | num) // 设置某一位为1
x & (~0 << n)          // 将x最右边的n位清零
x & ((1 << n) - 1)     // 将最高位到i位（含）清零
n & (n - 1) == 0       // 检查n是否为2的某次方，或者n是否为0

// 统计正数c二进制位1的个数
int count = 0;
for (; c != 0; c = c & (c - 1)) {
    count++;
}

// 变量交换
a ^= b;
b ^= a;
a ^= b;

// if(x == a) x = b; if(x == b) x = a;
x = a ^ b ^ x;

dp[i & 1]   //0,1循环
// 保留最低位的1，其他位都变为0
flag &= (-flag)

// 输出
String str = String.format("%02d",num);   // num不够两位则自动补全前面的0

// 判断两个int数相加是否溢出
if ((a > 0 && b > 0 && a > Integer.MAX_VALUE - b) 
  || (a < 0 && b < 0 && a < Integer.MAX_VALUE - b)) {
    // overflow
} 

// 判断一个元素至少出现两次
if (!once.add(key) && more.add(key)) {
  // Do Somthing
}
```

# Linux 命令
```bash
> fileName  #文件清空
```


>[常用位运算](http://blog.csdn.net/zmazon/article/details/8262185)



