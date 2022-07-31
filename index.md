## csapp lab2 bomb
### 总览
总共有六个炸弹，需要我们一一拆除。题目提供了可执行文件bomb与bomb.c，.c文件中只提供了主函数。
## phase_1
```markdown
/* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    phase_1(input);                  /* Run the phase               */
    phase_defused();                 /* Drat!  They figured it out!
				      * Let me know how they did it. */
printf("Phase 1 defused. How about the next one?\n");
 ```
接受输入后进行判断，判断通过会输出“Phase 1 defused. How about the next one?\n”。为了拆除炸弹，我们需要推理出input是什么。
命令详见书3.10.2
现在终端输入gdb bomb进行gdb调试
```markdown
root@cc661f098d2e:/csapp/2bomb# gdb bomb
(gdb) disas phase_1
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:     call   0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    call   0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    ret    
End of assembler dump.
```
根据注释<string_not_equal>易猜测：该函数为字符输入检测，在调用该函数前，给esi赋值，随后根据该函数的返回值判断是否执行explode_bomb爆炸，因此正确字符有可能存在于地址0x402400中。
```markdown
(gdb) x /s 0x402400
0x402400:       "Border relations with Canada have never been better."
```
发现“Border relations with Canada have never been better.”
接下来验证猜想
```markdown
Starting program: /csapp/2bomb/bomb 
warning: Error disabling address space randomization: Operation not permitted
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
```
正确！
## phase_2
反汇编phase_2得：
```markdown
(gdb) disas phase_2
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
   0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     call   0x40145c <read_six_numbers>
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    call   0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    call   0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
--Type <RET> for more, q to quit, c to continue without paging--disas 0x40145c
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:    add    $0x28,%rsp
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp
   0x0000000000400f42 <+70>:    ret    
End of assembler dump.

```
发现首次调用了：0x0000000000400f05 <+9>:     call   0x40145c <read_six_numbers>
反汇编该函数，得：
```markdown
(gdb) disas 0x40145c
Dump of assembler code for function read_six_numbers:
   0x000000000040145c <+0>:     sub    $0x18,%rsp
   0x0000000000401460 <+4>:     mov    %rsi,%rdx
   0x0000000000401463 <+7>:     lea    0x4(%rsi),%rcx
   0x0000000000401467 <+11>:    lea    0x14(%rsi),%rax
   0x000000000040146b <+15>:    mov    %rax,0x8(%rsp)
   0x0000000000401470 <+20>:    lea    0x10(%rsi),%rax
   0x0000000000401474 <+24>:    mov    %rax,(%rsp)
   0x0000000000401478 <+28>:    lea    0xc(%rsi),%r9
   0x000000000040147c <+32>:    lea    0x8(%rsi),%r8
   0x0000000000401480 <+36>:    mov    $0x4025c3,%esi
   0x0000000000401485 <+41>:    mov    $0x0,%eax
   0x000000000040148a <+46>:    call   0x400bf0 <__isoc99_sscanf@plt>
   0x000000000040148f <+51>:    cmp    $0x5,%eax
   0x0000000000401492 <+54>:    jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:    call   0x40143a <explode_bomb>
   0x0000000000401499 <+61>:    add    $0x18,%rsp
   0x000000000040149d <+65>:    ret    
End of assembler dump.

```
发现固定地址0x0000000000401480 <+36>:    mov    $0x4025c3,%esi
查看
```markdown
(gdb) x /s 0x4025c3
0x4025c3:       "%d %d %d %d %d %d"
```
发现格式化字符放在此，且类型为int
结合函数名称<read_six_numbers>，可以猜出，该函数的作用为读入输入的前六个字符。
根据猜测进行验证。
首先看phase_2的反汇编代码，
```markdown
0x0000000000400efc <+0>:     push   %rbp
0x0000000000400efd <+1>:     push   %rbx
0x0000000000400efe <+2>:     sub    $0x28,%rsp
```
该函数开始时，<+0>< +1>将rbp rbx的值放入栈，<+2>为该函数开辟空间

```markdown
0x0000000000400f3c <+64>:    add    $0x28,%rsp
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp

```
结束时，<+64>回收栈空间,<+68><+69>取回rbx，rbp的值。

```markdown
0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     call   0x40145c <read_six_numbers>

```
接着，将rsi指向栈rsp，调用函数<read_six_numbers>

```markdown
0x000000000040145c <+0>:     sub    $0x18,%rsp
   0x0000000000401460 <+4>:     mov    %rsi,%rdx
```
<+0>为该函数开辟空间，<+4>将rdx指向rsi。
此得，该函数传入了一个指针地址。


```markdown
0x0000000000401463 <+7>:     lea    0x4(%rsi),%rcx
   0x0000000000401467 <+11>:    lea    0x14(%rsi),%rax
   0x000000000040146b <+15>:    mov    %rax,0x8(%rsp)
   0x0000000000401470 <+20>:    lea    0x10(%rsi),%rax
   0x0000000000401474 <+24>:    mov    %rax,(%rsp)
   0x0000000000401478 <+28>:    lea    0xc(%rsi),%r9
   0x000000000040147c <+32>:    lea    0x8(%rsi),%r8

```
<+7>为rsi+4。
<+11>rsi+0x14。
<+20>rsi+0x10。
<+28>rsi+0xc。
<+32>rsi+0x8。
设rsi对应得的时num[0]，结合0x4025c3:       "%d %d %d %d %d %d"。则以上分别对应的是：num[1],num[5],num[4],num[3],num[2]。并且<+15>num[5]，<+24>num[4]放入栈。类型为int，int占4字节。

```markdown
0x0000000000401485 <+41>:    mov    $0x0,%eax
0x000000000040148a <+46>:    call   0x400bf0 <__isoc99_sscanf@plt>
   0x000000000040148f <+51>:    cmp    $0x5,%eax
   0x0000000000401492 <+54>:    jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:    call   0x40143a <explode_bomb>
   0x0000000000401499 <+61>:    add    $0x18,%rsp
   0x000000000040149d <+65>:    ret    

```
接下来，<+41>将rax置0，<+46>调用<__isoc99_sscanf@plt>,<+54>判断eax是否大于5。<+56>若大于，<+61>则释放空间,返回<+65>;否则，<+56>调用<explode_bomb>，爆炸。由此，sscanf函数返回了输入字符串的个数，因此推断出，我们输入的数（因为类型为int）必须大于等于6个。
因此可以大致推出read_six_number函数的原型等价为
```markdown
Void read_six_number(char *receive,int *num){
Char *style=“%d %d %d %d %d %d”
	 int re=sscanf（receive,style,&num[0] ,&num[1] ,&num[2] ,&num[3] ,&num[4] ,&num[5] ,&num[6]）;
if （re<5）
	explode_bomb（）；
}
```

返回phase_2
```markdown
0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    call   0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    call   0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
--Type <RET> for more, q to quit, c to continue without paging--disas 0x40145c
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
```
此时rsp的栈为
![image1](https://github.com/1051690662/csapp_lab/blob/gh-pages/11.jpg?raw=true)
<+14>若rsp指向的内存值为1，<+52>则跳转，否则，<+20>爆炸。

```markdown
0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp

```
此时，指向为
![image2](https://github.com/1051690662/csapp_lab/blob/gh-pages/22.jpg?raw=true)
注：c中不检查数组下标越界，因此rbp可以指向num[6]。

```markdown
0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    call   0x40143a <explode_bomb>
```
<+27>Eax=num[0]
<+30>Eax=2*num[0]
<+32>if（eax==(rbx)  2*num[0]==num[1]）
<+34>相同，goto<+41>
 	Else	
		<+36>不同，爆炸

```markdown
0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>

```
<+41>rbx指向下一个num[2]
<+45>if（rbp==rbx  当前rbp是否指向末尾num[6]？）
			<+50>Goto <+64>结束判断，跳出循环
Else	
<+48>Goto <+27>·
依次循环，综上可逆向出phase_2源码等价为：
```markdown
Void phase_2(char* receive) {
	Int num[6]=receive;
	Read_six_number(receive,num);
I	nt *start=&num[0];
	Int *end=&num[6]
	If(num[0]!=1)
		explode_bomb();
	While(start<end){
		If ((*start)*2!=*(start+1))
			Explode_bomb();
		else
			Start++;
	}
}
```		
以上跟符合反汇编的思路，可简化为：
```markdown
Void phase_2(char* receive) {
	Int num[6]=receive;
	Read_six_number(receive,num);
	For(int i=0;i<6;i++)
		If(num[i+1]!=num[i]*2)
			explode_bomb();
}
```
num[0]=1,后面是前面的2倍，因此答案为1 2 4 8 16 32。

## phase_3
```
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:     sub    $0x18,%rsp
   0x0000000000400f47 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000400f4c <+9>:     lea    0x8(%rsp),%rdx
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    call   0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    call   0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmp    *0x402470(,%rax,8)
   0x0000000000400f7c <+57>:    mov    $0xcf,%eax
   0x0000000000400f81 <+62>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:    mov    $0x2c3,%eax
   0x0000000000400f88 <+69>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:    mov    $0x100,%eax
   0x0000000000400f8f <+76>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:    mov    $0x185,%eax
--Type <RET> for more, q to quit, c to continue without paging--  
   0x0000000000400f96 <+83>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:    mov    $0xce,%eax
   0x0000000000400f9d <+90>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:    mov    $0x2aa,%eax
   0x0000000000400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:    mov    $0x147,%eax
   0x0000000000400fab <+104>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:   call   0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:   mov    $0x0,%eax
   0x0000000000400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:   mov    $0x137,%eax
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   call   0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   ret    
End of assembler dump.

```
此题与phase_2类似，<+0>开辟空间，传进来的两个值存放在栈中，用<+4>rcx，<+9>rdx指向这两个值。<+14>发现常地址，查看 
```
(gdb) x /s 0x4025cf
0x4025cf:       "%d %d"

0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    call   0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    call   0x40143a <explode_bomb>


```
结合<+19>给eax置0，调用<+24>sscanf函数，<+29>sscanf函数返回的值存储在eax中，与1比较。<+34>若小于1爆炸，否则继续。发现与phase_2套路一致，取传入的数需要大于两个，且只取前两个int类型数。

```
0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmp    *0x402470(,%rax,8)
0x0000000000400fad <+106>:   call   0x40143a <explode_bomb>

```
<+39>将第一个数的值与7比，<+106>若大于爆炸，<+46>否则继续，将第一个数的值放入eax。因此第一个数值的范围为0-7。若小于0就往回跳了
<+50>跳转到将eax的值*8加上地址0x402470后得到的新地址所对应的内存值。例若eax=1，则新地址为0x402478.查看其对应的内存值：
```
(gdb) x /a 0x402478
0x402478:       0x400fb9 <phase_3+118>

```
即跳转到0x400fb9
```
0x0000000000400fb9 <+118>:   mov    $0x137,%eax
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   call   0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   ret    

```
<+118>将0x137赋值给eax，<+123>比较传入的第二个数是否与其相等,若相等，<+134>则释放函数空间，<+138>返回；<+129>否则爆炸。因此，其中一组答案为：
1 311
如法炮制，所有答案有：0 207；1 311；2 707；3 256；4 389；5 206；6 682；7 327；

## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/1051690662/csapp_lab/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/1051690662/csapp_lab/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
