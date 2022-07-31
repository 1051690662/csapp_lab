## csapp lab2 bomb 《深入理解计算机系统》0基础超详细解析
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

## phase_4
```
(gdb) disas phase_4
Dump of assembler code for function phase_4:
   0x000000000040100c <+0>:     sub    $0x18,%rsp
   0x0000000000401010 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000401015 <+9>:     lea    0x8(%rsp),%rdx
   0x000000000040101a <+14>:    mov    $0x4025cf,%esi
   0x000000000040101f <+19>:    mov    $0x0,%eax
   0x0000000000401024 <+24>:    call   0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000401029 <+29>:    cmp    $0x2,%eax
   0x000000000040102c <+32>:    jne    0x401035 <phase_4+41>

   0x000000000040102e <+34>:    cmpl   $0xe,0x8(%rsp)
   0x0000000000401033 <+39>:    jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:    call   0x40143a <explode_bomb>

   0x000000000040103a <+46>:    mov    $0xe,%edx
   0x000000000040103f <+51>:    mov    $0x0,%esi
   0x0000000000401044 <+56>:    mov    0x8(%rsp),%edi
   0x0000000000401048 <+60>:    call   0x400fce <func4>

   0x000000000040104d <+65>:    test   %eax,%eax
   0x000000000040104f <+67>:    jne    0x401058 <phase_4+76>
   0x0000000000401051 <+69>:    cmpl   $0x0,0xc(%rsp)
   0x0000000000401056 <+74>:    je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:    call   0x40143a <explode_bomb>
--Type <RET> for more, q to quit, c to continue without paging--
   0x000000000040105d <+81>:    add    $0x18,%rsp
   0x0000000000401061 <+85>:    ret    
End of assembler dump.

```
基本套路与phase_2一致。
<+0>~<+32>易知（详见phase_2），传入两个int数。
<+34>~<+41>易知，第一个参数的值<15
<+46>~<+60>得，edx=15，esi=0；edi存储传入的第一个数，也就是需要我们求的数。随后调用函数<func4>
反汇编<func4>
```
(gdb) disas func4  

Dump of assembler code for function func4:
   0x0000000000400fce <+0>:     sub    $0x8,%rsp
   0x0000000000400fd2 <+4>:     mov    %edx,%eax
   0x0000000000400fd4 <+6>:     sub    %esi,%eax
   0x0000000000400fd6 <+8>:     mov    %eax,%ecx

   0x0000000000400fd8 <+10>:    shr    $0x1f,%ecx
   0x0000000000400fdb <+13>:    add    %ecx,%eax
   0x0000000000400fdd <+15>:    sar    %eax

   0x0000000000400fdf <+17>:    lea    (%rax,%rsi,1),%ecx
   0x0000000000400fe2 <+20>:    cmp    %edi,%ecx
   0x0000000000400fe4 <+22>:    jle    0x400ff2 <func4+36>
   0x0000000000400fe6 <+24>:    lea    -0x1(%rcx),%edx
   0x0000000000400fe9 <+27>:    call   0x400fce <func4>
   0x0000000000400fee <+32>:    add    %eax,%eax
   0x0000000000400ff0 <+34>:    jmp    0x401007 <func4+57>
   0x0000000000400ff2 <+36>:    mov    $0x0,%eax

   0x0000000000400ff7 <+41>:    cmp    %edi,%ecx
   0x0000000000400ff9 <+43>:    jge    0x401007 <func4+57>
   0x0000000000400ffb <+45>:    lea    0x1(%rcx),%esi
   0x0000000000400ffe <+48>:    call   0x400fce <func4>

   0x0000000000401003 <+53>:    lea    0x1(%rax,%rax,1),%eax
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000000000401007 <+57>:    add    $0x8,%rsp
   0x000000000040100b <+61>:    ret    
End of assembler dump.
	
```
<+4>Eax=edx=15,
<+6>eax=eax-esi=15-0=15;
<+8>ecx=eax=15
<+10>ecx=ecx>>31；
<+13>eax=eax+ecx=15+0=15；
<+15>eax=eax>>1=7
<+10>~<+15>等价为 eax=15，ecx=15的初值下，执行：eax=((ecx>>31)+eax)>>1，eax=7
<+17>ecx=1*rsi+rax=0+7=7
<+20>将传入的edi与ecx比较，若大于，则执行<+24>后执行<+27>func4函数，此处应该是个递归调用，具体含义不明，但是可以确定的是若想继续执行完成该函数，调用一定次数后们一定有edi<=ecx，于是跳转到<+36>。
<+41>~<+48>同上述原理，一定有ecx>=edi时，跳转到<+57>结束函数。
综合上述，当edi=ecx时，func4函数调用结束。因此edi为7；
此后返回函数phase_4
<+65>~<+85>易知：第二个参数为0。
故本题答案为7 0。
	
## phase_5
```
(gdb) disas phase_5
Dump of assembler code for function phase_5:
   0x0000000000401062 <+0>:     push   %rbx
   0x0000000000401063 <+1>:     sub    $0x20,%rsp
   0x0000000000401067 <+5>:     mov    %rdi,%rbx
   0x000000000040106a <+8>:     mov    %fs:0x28,%rax
   0x0000000000401073 <+17>:    mov    %rax,0x18(%rsp)
   0x0000000000401078 <+22>:    xor    %eax,%eax
   0x000000000040107a <+24>:    call   0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    call   0x40143a <explode_bomb>
   0x0000000000401089 <+39>:    jmp    0x4010d2 <phase_5+112>
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx
   0x000000000040108f <+45>:    mov    %cl,(%rsp)

   0x0000000000401092 <+48>:    mov    (%rsp),%rdx
   0x0000000000401096 <+52>:    and    $0xf,%edx

   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
--Type <RET> for more, q to quit, c to continue without paging--
   0x00000000004010ae <+76>:    movb   $0x0,0x16(%rsp)
   0x00000000004010b3 <+81>:    mov    $0x40245e,%esi
   0x00000000004010b8 <+86>:    lea    0x10(%rsp),%rdi
   0x00000000004010bd <+91>:    call   0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:    test   %eax,%eax
   0x00000000004010c4 <+98>:    je     0x4010d9 <phase_5+119>
   0x00000000004010c6 <+100>:   call   0x40143a <explode_bomb>
   0x00000000004010cb <+105>:   nopl   0x0(%rax,%rax,1)
   0x00000000004010d0 <+110>:   jmp    0x4010d9 <phase_5+119>
   0x00000000004010d2 <+112>:   mov    $0x0,%eax
   0x00000000004010d7 <+117>:   jmp    0x40108b <phase_5+41>
   0x00000000004010d9 <+119>:   mov    0x18(%rsp),%rax
   0x00000000004010de <+124>:   xor    %fs:0x28,%rax
   0x00000000004010e7 <+133>:   je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:   call   0x400b30 <__stack_chk_fail@plt>
   0x00000000004010ee <+140>:   add    $0x20,%rsp
   0x00000000004010f2 <+144>:   pop    %rbx
   0x00000000004010f3 <+145>:   ret    
End of assembler dump.

```
开头与前几题一样，在<+8>设置了堆栈金丝雀保护。
<+24>调用了< string_length >函数，根据函数名，与<+29>得：输入的是6个字符。<+34>否则爆炸。接下来来到了<+112>将eax置零后来到了<+41>。
根据<+5>得rdi接受传入的第一个字符的地址，且rbx指向其。<+41>在此条开始之前，eax被置零了（+112），mov指令因此ecx=（rbx）。<+45>将ecx的低8位保存在rsp所指向的栈中。<+48>将该值(ecx的低8位)赋给rdx。<+52>获得rdx的低4位赋给edx。<+55>查看地址常量0x4024b0，发现如下：
```
(gdb) x /s 0x4024b0
0x4024b0 <array.3449>:  "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"

```
经历了<+52>的操作后，rdx∈【0，15】，因此<+55>中，地址范围在0x4024b0~0x4024bf， 对应的edx值为“maduiersnfotvbyl“中的一个，取决于rdx的值。分析<+62>~<+74>发现这是一个循环，计数器为rax，总共执行六次。<+62>把edx的低8位存入栈，构建为一个数组。<+76>执行六次后在字符串末尾填“0”。<+81>查看常地址0x40245e，
	
```
(gdb) x /s 0x40245e
0x40245e:       "flyers"

```	
发现esi存储flyers
<+86>将栈中首字母的地址赋给rdi
<+91>随后调用函数<strings_not_equal>,结合<+96>~<+100>与函数名和前几题得经验分析得，该函数将判定栈中存储得值是否为esi中的flyers，返回一个值，若是则继续，否则爆炸。
因此，结合上述所有分析，phase_5的整个过程为输入6个字符，取每个字符的低8位与0xf位与后，得到一个0-15的范围，将地址0x4024b0加上该值索引得到的值为flyers。而地址0x4024b0~0x4024bf对应的字符为“ maduiersnfotvbyl”，要从中取出flyers，那么传入的原始数据的低8位分别为102，108，121，101，114，115，即可。若只讨论输入的是英文，根据ascii码查表得，答案只需要满足以下条件，：
第一个字符：I Y i y 
第二个字符：O o 
第三个字符：N n
第四个字符：E U e u
第五个字符：F V f v
第六个字：G W g w
只需将上述每个字符中任选一个，拼合即为正确答案。如：ionefg为一个答案。

## phase_6
```
	Phase_6
(gdb) disas phase_6 
Dump of assembler code for function phase_6:
   0x00000000004010f4 <+0>:     push   %r14
   0x00000000004010f6 <+2>:     push   %r13
   0x00000000004010f8 <+4>:     push   %r12
   0x00000000004010fa <+6>:     push   %rbp
   0x00000000004010fb <+7>:     push   %rbx
   0x00000000004010fc <+8>:     sub    $0x50,%rsp

   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    call   0x40145c <read_six_numbers>
   0x000000000040110b <+23>:    mov    %rsp,%r14

   0x000000000040110e <+26>:    mov    $0x0,%r12d
   0x0000000000401114 <+32>:    mov    %r13,%rbp
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax
   0x000000000040111b <+39>:    sub    $0x1,%eax
   0x000000000040111e <+42>:    cmp    $0x5,%eax
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    call   0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d
   0x000000000040112c <+56>:    cmp    $0x6,%r12d
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000000000401132 <+62>:    mov    %r12d,%ebx
   0x0000000000401135 <+65>:    movslq %ebx,%rax
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    call   0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>

   0x000000000040116f <+123>:   mov    $0x0,%esi
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx
   0x000000000040117a <+134>:   add    $0x1,%eax
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   0x000000000040118d <+153>:   add    $0x4,%rsi
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx
   0x000000000040119a <+166>:   cmp    $0x1,%ecx
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>
   0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:   mov    %rbx,%rcx
--Type <RET> for more, q to quit, c to continue without paging--
   0x00000000004011bd <+201>:   mov    (%rax),%rdx
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)
   0x00000000004011c4 <+208>:   add    $0x8,%rax
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
   0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
   0x00000000004011da <+230>:   mov    $0x5,%ebp
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:   mov    (%rax),%eax
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   call   0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
   0x00000000004011f7 <+259>:   add    $0x50,%rsp
   0x00000000004011fb <+263>:   pop    %rbx
   0x00000000004011fc <+264>:   pop    %rbp
   0x00000000004011fd <+265>:   pop    %r12
--Type <RET> for more, q to quit, c to continue without paging--
   0x00000000004011ff <+267>:   pop    %r13
   0x0000000000401201 <+269>:   pop    %r14
   0x0000000000401203 <+271>:   ret    
End of assembler dump.

```
第一部分<+0>~<+95>
```
	(gdb) disas phase_6 
Dump of assembler code for function phase_6:
   0x00000000004010f4 <+0>:     push   %r14
   0x00000000004010f6 <+2>:     push   %r13
   0x00000000004010f8 <+4>:     push   %r12
   0x00000000004010fa <+6>:     push   %rbp
   0x00000000004010fb <+7>:     push   %rbx
   0x00000000004010fc <+8>:     sub    $0x50,%rsp

   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    call   0x40145c <read_six_numbers>
   0x000000000040110b <+23>:    mov    %rsp,%r14

   0x000000000040110e <+26>:    mov    $0x0,%r12d
   0x0000000000401114 <+32>:    mov    %r13,%rbp
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax
   0x000000000040111b <+39>:    sub    $0x1,%eax
   0x000000000040111e <+42>:    cmp    $0x5,%eax
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    call   0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d
   0x000000000040112c <+56>:    cmp    $0x6,%r12d
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000000000401132 <+62>:    mov    %r12d,%ebx
   0x0000000000401135 <+65>:    movslq %ebx,%rax
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    call   0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi

```
②根据前面的经验，易知<+18>本题输入6个int数，假设存储的数组名为num
<+12>~<+23>:将r13,rsi,r14,rbp都指向了初始位置，即num[0]的地址。
①	根据<+26><+52><+56>易知：r12是一个循环的计数器，且循环总共执行6次。
接下来观察，这个循环做了什么。<+35>~<+45>判断当前值（num[r12]）的大小是满足num[0]-1<=5，即num[0]是否<=6,>=1（jbe为无符号数的比较，因此初值不能为负，比较前有减一操作，因此最小为1，不可为0；若为0，减1后会是tmax，不满足条件）。
若该循环次数小于6次，就来到了<+62>处。（①）
<+62>~<+65>ebx=r12,rax=ebx。
③	<+68>eax=*(rsp+rax*4)。rsp为数组首地址，rax为偏移数，int类型空间为4字节，因此索引时需rax*4，因此eax=num[ebx]此处为什么不直接用与之等价的num[r12]?我们接着往下看
<+71>与eax与（rbp）比较，num[r12]与*rbp进行比较（②），如果相等就爆炸。④此处为什么不用与rbp等价的rsp？我们接着看。代码来到了<+81>ebx++,
<+84>~<+87>可知，ebx作为这一小循环（<+65>~<+84>）的计数器，而每做一次，ebx的值就加一，rax作为索引的偏移数也跟着加一，因此与eax比较的值一直在发生变化。在本小循环中eax的值为num[r12]，而与之比较的值为num[rax]，一直在变，循环每做一次都往后比一个，直到数组末尾num[6](c不检查数组越界，因此可以索引到6)跳出。所以该小循环的作用是确保num[r12]（当前值）后的值不与其相等（num[r12]!=num[r12~5]）。因此③处不能用num[r12]。
接下来，到了<+89>r13+=4，goto<+32>rbp=r13,因此rbp指向下一个，解释了④。
随后重复上述循环。
综上，此处代码等价为以下c语言。其中i=r12；j=ebx。⑤处的num[i]用num[r13]索引，⑥处的num[i]用rsp+rax*4索引，num[j]用rbp索引。
```
for(int i=0;i<6;i++){
if(0<num[i] && num[i]<6){⑤
	for(int j=i;j<6;j++)
		if(num[i]==num[j])⑥
			explode_bomb();
}
else
explode_bomb();
}
```
第二部分
<+95>~<+121>
```
 0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>	
```	
这一部分比较容易，不赘述。过程等价为：
```	
for(int i=0;i<6;i++)
	num[i]=7-num[i];
```
接下来到了第三部分，比较繁琐，也是本题的核心，有一定的理解难度（不知道结果的情况下）。
<+123>~<+181>
```
 0x000000000040116f <+123>:   mov    $0x0,%esi
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx
   0x000000000040117a <+134>:   add    $0x1,%eax
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   0x000000000040118d <+153>:   add    $0x4,%rsi
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx
   0x000000000040119a <+166>:   cmp    $0x1,%ecx
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>	
	```
	<+123>:将esi置0后来到了<+163>ecx=*(rsi*1+rsp)，因为rsp为num的首地址，第一趟时rsi被置为0，所以等价于ecx=num[0]，<+166>ecx==1？假设num[0]==1，则来到了<+143>edx=0x6032d0。此值类似一个地址。
<+148>*（0x20+rsp+rsi*2）=rdx根据上一部分的经验，rsp是基地址，rsi是偏移量，又添加了0x20的常偏移量，因此这是一个新的数组，我们假设数组名为dizhi，我们查看该地址。<+153>~<+157>是否到达dizhi末，即num是否被遍历完。若遍历完该部分结束。
随后来到了<+163>ecx=rsp+rsi*1，此处更新了ecx的值，即ecx=num[rsi]，<+166>与1比较，现在我们讨论当num[rsi]不等于1时发生了什么。<+171>eax=1，<+176>edx=0x6032d0设置完两个初值后，来到了<+130>此处rdx=*（rdx+8）
我们查看地址rdx+8，即0x6032d0+8=0x6032d8
```
	(gdb) x/a 0x6032d8
0x6032d8 <node1+8>:     0x6032e0 <node2>
```
	发现他的值为0x6032e0，似乎也是一个地址，也就是说此时的rdx=0x6032e0。
<+134>~<+139>分析得，每循环一次eax的值就+1，rdx再次重复上述操作，即新的rdx=将原rdx的值+8后作为地址去解引。直到ecx=eax，跳转到<+148>将dizhi[rsi]=rdx。在第二轮时，
```
	(gdb) x/a 0x6032e8
0x6032e8 <node2+8>:     0x6032f0 <node3>

```
	我们发现，rdx+8再索引总是对应了一个新的地址值，因此我们试着尝试查看最初0x6032d0附近的值
```
	⑥(gdb) x /25w 0x6032d0
0x6032d0 <node1>:       0x14c   0x1     0x6032e0 <node2>        0x0
0x6032e0 <node2>:       0xa8    0x2     0x6032f0 <node3>        0x0
0x6032f0 <node3>:       0x39c   0x3     0x603300 <node4>        0x0
0x603300 <node4>:       0x2b3   0x4     0x603310 <node5>        0x0
0x603310 <node5>:       0x1dd   0x5     0x603320 <node6>        0x0
0x603320 <node6>:       0x1bb   0x6     0x0     0x0
0x603330:       0x0

```
我们以一字的间隔去解释存储值时发现，该结构存储了2个int和一个地址，而存储的地址又恰好指向下一个节点的地址。该结构为链表。
数据结构为 
```
typedef struct Link{
Int val；
Int num；
struct Link* next；
}；
Link node[5];
```
node[1].val=0x14c；node[1].num=1；node[1].next=0x6032e0。
Node[2].val=0xa8;node[2].num=2;node[2].next=0x6032f0
其中node[2].val的地址为：0x6032e0
node[2].next的地址为：0x6032e8
依次类推
发现，rdx存储的是指向某个节点的地址。
因此与该部分作用等价的伪代码为
```
Node[]={0x6032d0,0x6032e0,0x6032f0,0x603300,0x603310,0x603320};
for(int i=0;i<6;i++)
	if(num[i]==1)
		dizhi[i]==0x6032d0;
	else
		dizhi[i]=node[num[i]-1]
```
因此num[i]的值为x，则dizhi[i]的值为第x个节点的地址。
该部分结束。
		     
来到了第四部分,该部分比较考察基本功且较抽象，但是一步步画出栈即可轻松理解。
<+183>~<+220>
```
	0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:   mov    %rbx,%rcx
--Type <RET> for more, q to quit, c to continue without paging--
   0x00000000004011bd <+201>:   mov    (%rax),%rdx
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)
   0x00000000004011c4 <+208>:   add    $0x8,%rax
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
```
	dizhi数组的值分别指向一个节点的地址，而节点的地址+8就是存储该节点next指针的地址。该部分的作用是，将dizhi[i]所对应的节点的next指针，指向dizhi[i+]
等价伪代码为：
```
For(int i=0;i<6;i++)
	*(dizhi[i]+8)=dizhi[i+1]
```
一直到此，输入的值究竟是什么仍未有头绪。
		     
第五部分<+222>~< +257>
```
	0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
   0x00000000004011da <+230>:   mov    $0x5,%ebp
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:   mov    (%rax),%eax
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   call   0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
```
这一部分比较简单。简述
<+222>末尾节点的next设为NULL（0）
而rbx一直是dizhi[0]的值未改变，该部分的作用就是从索引节点地址=dizhi[0]的节点，将该节点的val与下一节点的val比较，若大于则继续，否则爆炸。
因此等价该部分的伪代码为
```
For(Link* now=dizhi[0]；now；now=now->next)
If (now.val > now.next.val)
	Continue;
else
	explode_bomb();
}
```
到此，该题结束。

综上，输入可以逆推。要想不触发爆炸函数，第五部分的节点指向必须为节点值从大到小排列，根据⑥
<node1>0x6032d0 =0x14c
<node2>0x6032e0 =0xa8    
<node3>0x6032f0 =0x39c   
<node4>0x603300=0x2b3   
<node5>0x603310 =0x1dd   
<node6>0x603320 =0x1bb
因为0x39c>0x2b3>0x1dd>0x1bb>0x14c>0xa8。因此节点指向为：dizhi[0]=0x6032f0，0x6032f0=&node[3]，*（0x6032f8）=0x603300依次类推等价伪代码为：dizhi[0]=0x6032f0，&（node[3].next）=0x6032f8，node[3].next=&node[4]，node[4].next=&node[5] ，node[5].next=&node[6] ，node[6].next=&node[1] ，node[1].next=&node[2]。
因此dizhi[0]=&node[3]，dizhi[1]=&node[4] ，dizhi[2]=&node[5] ，dizhi[3]=&node[6] ，dizhi[4]=&node[1] ，dizhi[5]=&node[2]。
根据第三部分知道，dizhi数组的值是通过以num数组的值做偏移量，在0x6032d0为基址不断解引得来的。因此num[i]的值决定了dizhi[i]存储的是第几个node。因此num[0]=3，num[1]=4，num[2]=5，num[3]=6，num[4]=1，num[5]=2。到此为止，仍不是最初答案，该部分是经历了第二部分后的值，所以要还原最初的值，还需要用7减。因此最开始的值为num[6]={4,3,2,1,6,5}。
故最终答案为4 3 2 1 6 5。




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
