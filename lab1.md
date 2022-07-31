总体概览

![image1](https://github.com/1051690662/csapp_lab/blob/gh-pages/33.png?raw=true)

第一部分，integer

要求
```markdown
1. Integer constants 0 through 255 (0xFF), inclusive. You are
      not allowed to use big constants such as 0xffffffff.
  2. Function arguments and local variables (no global variables).
  3. Unary integer operations ! ~
  4. Binary integer operations & ^ | + << >>
    
  Some of the problems restrict the set of allowed operators even further.
  Each "Expr" may consist of multiple operators. You are not restricted to
  one operator per line.
 
  You are expressly forbidden to:
  1. Use any control constructs such as if, do, while, for, switch, etc.
  2. Define or use any macros//��.
  3. Define any additional functions in this file.
  4. Call any functions.
  5. Use any other operations, such as &&, ||, -, or ?:
  6. Use any form of casting//.
  7. Use any data type other than int.  This implies that you
     cannot use arrays, structs, or unions.
 
 
  You may assume that your machine:
  1. Uses 2s complement, 32-bit representations of integers.
  2. Performs right shifts arithmetically.
  3. Has unpredictable behavior when shifting if the shift amount
     is less than 0 or greater than 31.

```
可以使用：& ^ | + << >> ~ ！，常量0-255，int数据类型，操作数不能超过每题的max ops

不可以使用：任何控制语句：if while for switch等，宏，函数，&& &#124;| - ？等，数组，结构体，除int外的任何数据类型

题目

1.int bitXor（int x，int y）

题意：使用~！对xy实现位级的异或运算

```markdown
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
 
  return (~(x & y)) & (~(~x & ~y));
}
```

解：

列出xy真值表

x y result

0 0 0

0 1 1

1 0 1

1 1 1

离散数学易得：（x|y）对应情况1~3

~（x&y）对应4

列出真值表发现：（x|y）&~（x&y）符合要求，按题意运算符等价得：

(~(~x&~y））&(~（x&y）)

2.int tmin(void)
题意：返回二进制最小数

```markdown
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
 
  return 0x1<<31;
 
}
```

解：32位int，二进制最小数Tmin=0x10000000。根据题目要求不能直接定义超过255的int，故用移位实现0x1<<31

3.int isTmax(int x)

题意：如果x是最大二进制数0x7fffffff，返回1；否则，返回0。
```markdown
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
return (!(!(x+1)))&(!((x+1)^(~x)));
}
```
解：

发现：

Tmax+1=Tmin

~Tmax=Tmin

得：!((x+1)^(~x))

可是当x=-1时，也满足上式，故需额外排除此情况，当x+1为0时，输出0。没有&&因此x+1==0等价为！（！（x+1））。最终合并后得答案为：(!(!(x+1)))&(!((x+1)^(~x)))

4.int negate(int x)

题意：返回x的负数
```markdown
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```
解：二进制负数为取反加一。即：~x+1。或减一取反，即~（x-1）

5.int allOddBits(int x)

题意：如果x二进制的奇数位都为1，则返回1
```markdown
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
    int f = 0xaa;
          f=(f<<8)+f;
          f=(f<<16)+f;
  return !((x&f)+(~f+1));
}
```
解：

注意！他的最低为为0，最高位为31，所以只要满足x&0xffffffff在0xaaaaaaaa位上都有1,即满足题意。因此得：（x&f）-f==0时，输出1。二进制尺度上，负号=~f+1，因此答案为：!((x&f)+(~f+1))

由于不能直接定义大于255得常数，因此需位移扩展由f=0xaa；f=(f<<8)+f;

          f=(f<<16)+f;得f=0xaaaaaaaa。

6.int isAsciiDigit(int x)

题意：如把x为ascii码编排下的0-9，0x30 <= x <= 0x39，返回1
```markdown
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  return !(x + ~0x30+1 >> 31) & !(0x39 + ~x+1 >> 31);
}
```
解：

题意得：x-0x30>=0 &&0x39-x>=0

结果若大于0，二进制开头为0，故答案为：!(x + ~0x30+1 >> 31) & !(0x39 + ~x+1 >> 31)

7.int conditional(int x, int y, int z)

题意：如果x！=0返回y，否则返回z
```markdown
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
 int f=~(!x)+1;
return ~f&y|(f&z);
}
```
解：

x=true（！=0）时，!x=0，~（！x）=0xffffffff，加1后，f=0，输出y：~f&y。同理，x=False(=0)，f=0xffffffff，输出z： f&z。

合并后答案为：~f&y|(f&z)

8.int isLessOrEqual(int x, int y)

题意：如果x<y.返回1；否则，返回0
```markdown
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int t=x+(~y+1);
  int ts=(t>>31)&1;
  int xs=(x>>31)&1;
  int ys=(y>>31)&1;
  int nxs=!xs;
  int nys=!ys;
  return (xs&nys)|(nxs&nys&ts)|(xs&ys&ts)|(!t);
}
```
解：

根据离散数学，列出对应的符号位真值表：

x y x-y result

0 0 0  0

0 0 1  1

0 1 0  0

0 1 1  0

1 0 0  1

1 0 1  1

1 1 0  0

1 1 1  1

不难发现，当1 0，0 0 1，1 1 1时返回1，易得(xs&nys)|(nxs&nys&ts)|(xs&ys&ts);

其中：t=x-y，ts=x-y的符号位，xs=x的符号位，nxs=x符号位的非；y同理。再加上两数相等的判定（！t），即为答案。

9.int logicalNeg(int x)

题意：不使用！实现逻辑非。
```markdown
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  int nx=~x+1;
  return ((~(x|nx))>>31)&1;
}
```
解：

考虑特殊的x，则其对应的二进制原码与补码（取反加1）为

0    Tmin        Tmax       -1

原码0x0  0x80000000  0x7fffffff     0xffffffff

补码0x0  0x80000000  0x80000001  0x00000001

化为二进制后发现，以上数的原码与补码的符号位构成

0 0；1 1；01；10四种情况，涵盖所有可能，画出对应真值表如下

原始数据 原码符号位 补码符号位 输出结果

0            0         0          1

Tmax         0         1          0

-1           1         0           0

Tmin         1         1           0

设原码为x，补码为nx

写出主析取范式（极小项）：（非x符号位 合取 非nx符号位）化简等价得：~(x|nx)>>31，

注意：算数右移时填充符号位，结果为0xffffffff，(若要实现逻辑右移，可强制转化为unsigned，可题目不允许使用非int数据类型)题目要求输出为布尔值（0|1），故需加上&1，最终答案为((~(x|nx))>>31)&1

10.int howManyBits(int x)

题意：x最少可以用几位二进制补码表示
```markdown
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  int s,code,y,h16,h8,h4,h2,h1;
  s=x>>31;
  code = (s & ~x) | (~s & x);
  
   y=code>>16;
  h16=!!y<<4;
  code=code>>h16;
 
  y=code>>8;
  h8=!!y<<3;
  code=code>>h8;
 
  y=code>>4;
  h4=!!y<<2;
  code=code>>h4;
 
  y=code>>2;
  h2=!!y<<1;
  code=code>>h2;
 
  y=code>>1;
  h1=!!y<<1;
  code=code>>h1;
 
  return h16+h8+h4+h2+h1+code+1;
}
```
解：

在计算机中，正数以原码存储，负数以补码存储。题目求原数据所需的补码的最小位数，则正数的求法为找到最高有效位，加上1位符号位，负数求得其原码（绝对值）的最高有效位后再加1位符号位。

注意：原码补码互化时用取反加一，而互化时，取反加一和减一取反等价。因此，除了-1（0xfff）外，取反后加不加1并不会影响有效位。相当于原码-1再取反，不-1数据表示的位数肯定大于减后的位数。因此取反加一的加一操作反而会造成-1的特例，变为0x00000001，那根据求法的话，-1就需要2位表示。事实上-1只需要1位即可表示，即1既做符号位又做数值位。因此，-1处理后应理想为0x00000000。所以在此为了处理-1不需要加一。只需要正数不变，负数取反即可。int s=x>>31;

int code = (s & ~x) | (~s & x);

统计最高位1的位置则用二分法就好啦。先向右位移16位，y=code>>16;若剩下的数不为0，即高16位至少有一位有效，则向右被移动（抹去）的16位有效，即最高位>=16，所需的最少位数也>=16，h16=!!y<<4，因此原始数据的左16位仍待判断最高有效位，因此原始数据需要更改，同样右移16位后继续下一步操作；否则，说明右移多了，最高有效位<16，因此原始数据不更改，进行下一步操作。code=code>>h16。以此类推，最后剩下最后一位，也就是code本身，把他们全部相加，再加上符号位所占的1位，即为答案：h16+h8+h4+h2+h1+code+1

第二部分，float

要求
```markdown
For the problems that require you to implement floating-point operations,
the coding rules are less strict.  You are allowed to use looping and
conditional control.  You are allowed to use both ints and unsigneds.
You can use arbitrary integer and unsigned constants. You can use any arithmetic,
logical, or comparison operations on int or unsigned data.
 
You are expressly forbidden to:
  1. Define or use any macros.
  2. Define any additional functions in this file.
  3. Call any functions.
  4. Use any form of casting.
  5. Use any data type other than int or unsigned.  This means that you
     cannot use arrays, structs, or unions.
  6. Use any floating point data types, operations, or constants.
```
可以使用：while for if switch 等控制语句 unsigned ，int ，<,>,==等

不可以使用：宏，函数，结构体，数组，联合等，除int ，unsigned外的类型

11.unsigned floatScale2(unsigned uf)

题意：返回2*f的值，传入和传出的数据都将按单精度浮点数的编码规则解释。
```markdown
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  unsigned exp=(uf>>23)& 0xff;
  unsigned frac = uf & 0x7fffff;
  unsigned re;
  if (exp==0xff)
    re=uf;
  else if(exp)
    exp++;
  else
    frac<<=1;
  re= (uf&0x80000000) | (exp<<23) | frac;
  return re;
}
```
解：

传入一个无符号整数，将它转化为二进制，其二进制的含义为单精度实数的编码规则（s+exp+frac）。将该数乘二后，同样以单精度实数的编码规则负载于无符号整数上输出。

考虑三种情况：

1：exp=0xff，为Nan，输出原数。

2： exp！=0&&exp！=0xff，即为规格化>=1，将其exp加1即为原数乘2。

3：exp==0&&frac。为非规格化<1将frac左移一位即为原数乘二，此处又有两种情况，若frac的最高位为1(>=0.5)，左移一位后，需将exp设为1，变为规格化数；若最高为不为1，左移一位即可。故两种情况可通过(exp<<23) | frac合并。

其中uf&0x80000000为符号位。

12.int floatFloat2Int(unsigned uf)

题意：将uf强制转化为int类型输出
```markdown
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
  int re,s, exp, frac, e;
  s=uf>>31;
  exp=(uf>>23)&0xff;
  e=exp-127;
  frac=uf&0x7fffff;
  if(e<0)//2^e,int最大为2^(32-1)
    re=0;
  else if(e==0)
    re=1;
  else if(e>31)
    re=0x80000000u;
  else if(e>23)
    re=frac<<(e-23);
  else
    re=frac>>(23-e);
  if(s)
    re=-re;
  return re
}
```
解：

要求将float转化为int，转化时，直接截掉小数部分，只保留float的·整数部分，分为以下几种情况。

Float小于0，int=0
Float的阶码真实值等于0，因含有隐藏1，int=1
Float的整数部分超出int的最大值，int=0x80000000
Float二进制的有效位数超过32-9位
Flaot二进制的有效位数不超过32-9位
对于以上，我们只需要取出阶码exp=(uf>>23)&0xff，算出他的真实值

e=exp-127  //（(2^(n-1）)-1）n为阶码的位数;

对于情况3，int为32位，1位为符号位，因此有效数据为31位，而e表示的是2^n位，因此当e>31时，int溢出

对于情况4，float的有效位数大于他的frac位数，因此转化为int时需要左移e-23（frac的位数）位。

对于情况5，float的有效位数小于他的frac位数，因此转化为int时需要右移23-e位，截取小数部分。

13.unsigned floatPower2(int x)

题意：返回2^x的值，返回的值按单精度编码规则解释。
```markdown
* /
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
  unsigned re;
  int exp=x+127;
  if(exp>=255)
    re=0x7f800000;
  else if(exp<=0)
    re=0;
  else
    re=exp<<23;
  return re;
 
}
```
解：

输出2^x的二进制表示（依附于无符号）

Exp(阶码表示)-127=x（真实值）

当exp（8位）为全1或更大时（exp>=255），输出+INF（无穷大），当exp（8位）为全0或更小时（exp<=0）输出0，其余情况exp不为0和不为全一，为规格化浮点数，frac为全0时，任有隐藏1，题目又是2的幂，因此将exp的值=二进制左移23位，将他放在浮点表示法的正确的位置即可。
