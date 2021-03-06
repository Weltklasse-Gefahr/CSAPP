# Table of Contents

=================



​	\* [phase_3](#phase_3)

​	\* [phase_4](#phase_4)

​	\* [phase_5](#phase_5)

​	\* [phase_6](#phase_6)

​	\* [总结](#总结)

​		\* [对于汇编语言](#对于汇编语言)

​		\* [技能](#技能)

# phase_3

```assembly
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
```

之前知道 `__isoc99_sscanf@plt` 为按格式化读取字符串并返回格式化后的字符个数，这里寄存器 `rsi` 放的是格式化字符串，打断点，得到格式为 `%d %d`，说明此次密码为两个数字，中间以空格间隔。该函数将格式化后的两个字符分别存放在`rdx`，`rcx`中指向的内存地址处，即`0x8(%rsp),0xc(%rsp)`。

若输入数字数量为1，则 bomb！数量正确则跳转至 `400f6a`。

So：两个数字，中间以空格间隔。

```assembly
400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
```

`ja`是无符号数>，如果`0x8(%rsp) > 7`，跳转至 `400fad`，即 bomb！否则，间接跳转到 `0x402470+8*%rax` 的位置上存储的地址：

So：第一个参数为<=7的无符号数，将第一个参数从1到7赋值，其对应的跳转地址使用`x/g`打印如下：

| arg1 | 跳转位置           | 对应 ar2  |
| ---- | ------------------ | --------- |
| 1    | 0x402478: 0x400fb9 | 0x137=311 |
| 2    | 0x402480: 0x400f83 | 0x2c3=707 |
| 3    | 0x402488: 0x400f8a | 0x100=256 |
| 4    | 0x402490: 0x400f91 | 0x185=389 |
| 5    | 0x402498: 0x400f98 | 0xce=206  |
| 6    | 0x4024a0: 0x400f9f | 0x2aa=682 |
| 7    | 0x4024a8: 0x400fa6 | 0x147=327 |

```assembly
400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
400fc2:	74 05                	je     400fc9 <phase_3+0x86>
400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
```

如果`0xc(%rsp)==%eax`，跳转至`0x400fc9`，否则，bomb!

So：表格中七组解任一组均可defuse。



# phase_4

```assembly
40100c:	48 83 ec 18          	sub    $0x18,%rsp
401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
40101f:	b8 00 00 00 00       	mov    $0x0,%eax
401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
401029:	83 f8 02             	cmp    $0x2,%eax
40102c:	75 07                	jne    401035 <phase_4+0x29>
40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
401033:	76 05                	jbe    40103a <phase_4+0x2e>
401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
```

现在对 `__isoc99_sscanf@plt` 函数已经很亲切了，第一个参数输入字符串，第二个参数格式化字符串（存放在%rsi），格式化后的字符依次存入`rdx`，`rcx`中指向的内存地址处，即`0x8(%rsp),0xc(%rsp)`。打断点，得到格式为 `%d %d`，说明此次密码为两个数字，中间以空格间隔。

如果输入数字数量不等于2，bomb！

`jbe`是无符号<=，如果`0x8(%rsp) <= 14`，直接跳转至 `40103a`，否则，原地 bomb!

So：两个数字， 中间以空格间隔。第一个参数为<=14的无符号数，即取值从1-14。

```assembly
40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
40103f:	be 00 00 00 00       	mov    $0x0,%esi
401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
401048:	e8 81 ff ff ff       	callq  400fce <func4>
40104d:	85 c0                	test   %eax,%eax
40104f:	75 07                	jne    401058 <phase_4+0x4c>
```

寄存器赋值：`%edx = 0xe`，`%esi = 0x0`，`%edi = 0x8(%rsp)`。

如果`%rax != 0x0`，跳转至 `401058`，即原地 bomb！

So：需要 `func4` 返回值 `%rax = 0x0` 。观察 `func4`

```assembly
400fce:	48 83 ec 08          	sub    $0x8,%rsp
400fd2:	89 d0                	mov    %edx,%eax
400fd4:	29 f0                	sub    %esi,%eax
400fd6:	89 c1                	mov    %eax,%ecx
400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
400fdb:	01 c8                	add    %ecx,%eax
400fdd:	d1 f8                	sar    %eax
400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
400fe2:	39 f9                	cmp    %edi,%ecx
400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
400fee:	01 c0                	add    %eax,%eax
400ff0:	eb 15                	jmp    401007 <func4+0x39>
400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
400ff7:	39 f9                	cmp    %edi,%ecx
400ff9:	7d 0c                	jge    401007 <func4+0x39>
400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
401007:	48 83 c4 08          	add    $0x8,%rsp
40100b:	c3                   	retq  
```

因为逻辑稍微复杂了点， 写了伪代码如下（傻了吧竟然有平平无奇的一堆算术运算之后竟然还藏了个递归 OJZ

```c
// 初始值：%edi-第一个参数，%esi = 0x0, %edx = 0xe
%eax = %edx - %esi;// 0x2
%ecx = %eax >> 0x1f;// 0x0
%eax = (%ecx + %eax) >> 1;// 0x7
%ecx = %rax + %rsi;// 0x7
if(%ecx <= %edi){
    %eax = 0x0;
    if(%ecx >= %edi) return %eax;
    else{
        %esi = %rcx + 1;
        func4();
        %eax = %rax + %rax + 0x1;
    }
}else{
    %edx=%rcx-0x1;
    func4();
    %eax = %eax + %eax;
    return %eax;
}
```

要使 `%eax=0` ，一种可能性是 `%ecx <= %edi` 同时 `%ecx >= %edi`，即，第一个参数等于7。为什么说是一种可能性呢？因为观察到条件分支的结构很像，可能为无限逼近答案

```assembly
401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
401056:	74 05                	je     40105d <phase_4+0x51>
401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
```

如果`0xc(%rsp)=0x0`，跳转至`40105d`，否则，原地 bomb！

So：第二个参数为0（猝不及防的简单 0.0

# phase_5

熟悉的 `string_length`：输入字符数为6，熟悉的 `string_not_equal`，要比较的字符串所在位置 `0x40245e`，打印出来，为 `flyers`。

```c
// %rdi-输入参数
%rbx = %rdi;
%eax = 0x0;
do{
    %ecx = (%rbx + %rax);// 输入参数存放地址，一次取1字节
	(%rsp) = %cl;
	%rdx = (%rsp);
	%edx = %edx & 0xf; // 除最低4位，其他位置零
	%edx = (%rdx + 0x4024b0);
	(%rsp + %rax + 0x10) = %dl;
	%rax = %rax + 0x1;
}while(%rax ! = 6)
(%rsp+0x16) = 0x0;
%esi = 0x40245e;
%rdi = (%rsp + 0x10);
string_not_equal();
if(%eax != 0) bomb!
else ...
```

对输入参数进行一系列的操作，简单来说就是每次取1字符，该字符除最低4位外其他位置置零，然后将值与 `0x4024b0 `相加，取出相加后的值所指向的字符，将该字符与对应位置的 `flyers` 的字符做比较，计算完这6个字符后，再调用 `string_not_equal` ，不一致就 bomb!

打印出 `0x4024b0` 处的字符：

```
"maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```

要从中择出 `flyers`依次对应的字符，应该是据初始位置偏移量`f:9`，`l:15`，`y:14`，`e:5`，`r:6`，`s:7`

但 ASCII 表中能输出的字符值>=32，所以，在保持最低4位不变，最高4位不为0的 ASCII 数都可符合要求

所以：对应输入参数为初始位置偏移量+16的倍数

|      | >=32                                                         |
| ---- | ------------------------------------------------------------ |
| f:9  | +16\*2=41=')', +16\*3=57='9', +16\*4=73='I', +16\*5=89='Y', +16\*6=105='i', +16\*7=121='y' |
| l:15 | +16\*2=47='/', +16\*3=63='?', +16\*4=79='O', +16\*5=95='_', +16\*6=111='o' |
| y:14 | +16\*2=46='.', +16\*3=62='>', +16\*4=78='N', +16\*5=94='^', +16\*6=110='n' |
| e:5  | +16\*2=37='%', +16\*3=53='5', +16\*4=69='E', +16\*5=85='U', +16\*6=101='e', +16\*7=117='u' |
| r:6  | +16\*2=38='&', +16\*3=54='6', +16\*4=70='F', +16\*5=86='V', +16\*6=102='f', +16\*7=118='v' |
| s:7  | +16\*2=39=''', +16\*3=55='7', +16\*4=71='G', +16\*5=87='W', +16\*6=103='g', +16\*7=119='w' |

以上组合，任选，俏皮一点，`y/nevg`

# phase_6

phase_6 汇编码怎么这么长啊= =

熟悉的 `read_six_numbers` ，读出的六个参数分别存放在 `0x0(%rsp), 0x4(%rsp), 0x8(%rsp), 0xc(%rsp), 0x10(%rsp), 0x14(%rsp) `，

```assembly
________________________   Part 1    ________________________
40110b:	49 89 e6             	mov    %rsp,%r14
40110e:	41 bc 00 00 00 00    	mov    $0x0,%r12d

Section_1:
401114:	4c 89 ed             	mov    %r13,%rbp
401117:	41 8b 45 00          	mov    0x0(%r13),%eax
40111b:	83 e8 01             	sub    $0x1,%eax
40111e:	83 f8 05             	cmp    $0x5,%eax
401121:	76 05                	jbe    401128 <phase_6+0x34>
401123:	e8 12 03 00 00       	callq  40143a <explode_bomb>
401128:	41 83 c4 01          	add    $0x1,%r12d
40112c:	41 83 fc 06          	cmp    $0x6,%r12d
401130:	74 21                	je     401153 <phase_6+0x5f>
401132:	44 89 e3             	mov    %r12d,%ebx

Section2:
401135:	48 63 c3             	movslq %ebx,%rax
401138:	8b 04 84             	mov    (%rsp,%rax,4),%eax
40113b:	39 45 00             	cmp    %eax,0x0(%rbp)
40113e:	75 05                	jne    401145 <phase_6+0x51>
401140:	e8 f5 02 00 00       	callq  40143a <explode_bomb>
401145:	83 c3 01             	add    $0x1,%ebx
401148:	83 fb 05             	cmp    $0x5,%ebx
40114b:	7e e8                	jle    401135 <phase_6+0x41>
40114d:	49 83 c5 04          	add    $0x4,%r13
401151:	eb c1                	jmp    401114 <phase_6+0x20>
```

先放弃思考，写出伪代码

```c
// %rsp,%r13,%rsi -第一个参数
%r14 = %rsp;
i = 0;

Section_1:
    %rbp = %r13; //
    %eax = %r13 - 1;
    if(%eax > 0x5)//无符号 >
        explode(); //要求每个参数：[0,5]
    else{
        i++;
        if(i == 6)
            goto Part_2;
        else
            j = i;
    }
Section_2:
    %rax = j;
    %eax = %rsp + 4 * %rax; // 依次取输入参数
    if(%rbp == %eax) //要求：与该参数不相同
       explode();
    j ++;
    if(j <= 5) 
        goto Section_2;
    %r13 += 4; //
	goto Section_1;
```

以上部分，要求每个参数取值$[1,6]$，且互不相同。

```assembly
________________________   Part 2    ________________________
  401153:	48 8d 74 24 18       	lea    0x18(%rsp),%rsi
  401158:	4c 89 f0             	mov    %r14,%rax
  40115b:	b9 07 00 00 00       	mov    $0x7,%ecx
  
  Section_1:
  401160:	89 ca                	mov    %ecx,%edx
  401162:	2b 10                	sub    (%rax),%edx
  401164:	89 10                	mov    %edx,(%rax)
  401166:	48 83 c0 04          	add    $0x4,%rax
  40116a:	48 39 f0             	cmp    %rsi,%rax
  40116d:	75 f1                	jne    401160 <phase_6+0x6c>
  40116f:	be 00 00 00 00       	mov    $0x0,%esi
  401174:	eb 21                	jmp    401197 <phase_6+0xa3>
  
  Section_2:
  401176:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40117a:	83 c0 01             	add    $0x1,%eax
  40117d:	39 c8                	cmp    %ecx,%eax
  40117f:	75 f5                	jne    401176 <phase_6+0x82>
  401181:	eb 05                	jmp    401188 <phase_6+0x94>
  
  Section_3:
  401183:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  
 Section_4:
  401188:	48 89 54 74 20       	mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:	48 83 c6 04          	add    $0x4,%rsi
  401191:	48 83 fe 18          	cmp    $0x18,%rsi
  401195:	74 14                	je     4011ab <phase_6+0xb7>
  
Section_5:
  401197:	8b 0c 34             	mov    (%rsp,%rsi,1),%ecx
  40119a:	83 f9 01             	cmp    $0x1,%ecx
  40119d:	7e e4                	jle    401183 <phase_6+0x8f>
  40119f:	b8 01 00 00 00       	mov    $0x1,%eax
  4011a4:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  4011a9:	eb cb                	jmp    401176 <phase_6+0x82>
```

先放弃思考，写出伪代码

```c
// %rsp,%r13,%r14,%rbp -第一个参数
%rsi = %rsp + 0x18;
%rax = %r14;

Section_1: //对每一个参数，进行7减操作
    %edx = 7 - (%rax);
	%rax += 4;
	if($rax != %rsi) 
        goto Section_1;
	i = 0;
	goto Section_5;

// 对于每一个参数，根据它的值，选取合适的 node 地址存放在%rsp+0x20的一段地址中
Section_2:
	now = set_array[j];
	j++;
	if(j == array[i])
        goto Section_2;
	goto Section_4;

Section 3:
	set_array[] = 0x6032d0;

Section_4:
	(%rsp + i + 0x20) = now;
	i++; //下一个参数
	if(i == 6)
        goto Part_3;

Section_5:
	if(array[i] <= 1)
        goto Section_3;
	j = 1;
	set_array[] = 0x6032d0;
	goto Section_2;
```

打印出 `0x6032d0` 处的值，为链表 {332,168,924,691,477,443}

| %rsp+0x20       | %rsp+0x28       | %rsp+0x30       | %rsp+0x38       | %rsp+0x40       | %rsp+0x48       |
| --------------- | --------------- | --------------- | --------------- | --------------- | --------------- |
| &node[array[1]] | &node[array[2]] | &node[array[3]] | &node[array[4]] | &node[array[5]] | &node[array[6]] |

```assembly
  4011ab: 48 8b 5c 24 20        mov    0x20(%rsp),%rbx
  4011b0: 48 8d 44 24 28        lea    0x28(%rsp),%rax
  4011b5: 48 8d 74 24 50        lea    0x50(%rsp),%rsi
  4011ba: 48 89 d9              mov    %rbx,%rcx

Section_1
  4011bd: 48 8b 10              mov    (%rax),%rdx
  4011c0: 48 89 51 08           mov    %rdx,0x8(%rcx)
  4011c4: 48 83 c0 08           add    $0x8,%rax
  4011c8: 48 39 f0              cmp    %rsi,%rax
  4011cb: 74 05                 je     4011d2 <phase_6+0xde>
  4011cd: 48 89 d1              mov    %rdx,%rcx
  4011d0: eb eb                 jmp    4011bd <phase_6+0xc9>

Section_2
  4011d2: 48 c7 42 08 00 00 00  movq   $0x0,0x8(%rdx)
  4011d9: 00 
  4011da: bd 05 00 00 00        mov    $0x5,%ebp
  
Section_3
  4011df: 48 8b 43 08           mov    0x8(%rbx),%rax
  4011e3: 8b 00                 mov    (%rax),%eax
  4011e5: 39 03                 cmp    %eax,(%rbx)
  4011e7: 7d 05                 jge    4011ee <phase_6+0xfa>
  4011e9: e8 4c 02 00 00        callq  40143a <explode_bomb>
  4011ee: 48 8b 5b 08           mov    0x8(%rbx),%rbx
  4011f2: 83 ed 01              sub    $0x1,%ebp
  4011f5: 75 e8                 jne    4011df <phase_6+0xeb>
  4011f7: 48 83 c4 50           add    $0x50,%rsp
```



```c
i = 0;
j = 1;
%rcx = i;

Section_1:
    array[j] = &array[j];
    j++;
    if(j == 5)
        goto Section_2;
    %rcx = array[j];
    goto Section_1;
Section_2:
    (%rdx+8)=0;
    n = 5;

Section_3:
    if(array[i] >= array[i+1]) // 必须为降序
        explode();
    i++;
    n--;
    if(n != 0)
        goto Section_3;
    return;
```

将node的值，按降序放置，即{924(node_3),691(node_4),477(node_5),443(node_6),332(node_1),168(node_2)}

顺序为 `3,4,5,6,1,2`

进行7减操作，要求输入参数为：`4 3 2 1 6 5`

# 总结

## 对于汇编语言

1. 想写出对应的伪代码结构，观察 `jmp` 等跳转指令是灵魂，在跳转的位置上分段，结合C 的 `goto`，可以很快的写出结构。
2. 观察一些 `+=1,2,4` 的寄存器，其往往充当的是循环变量 `i,j,k` 的作用，用这些符号全部替换之后可以简化代码。
3. 利用编辑器观察寄存器的使用情况（在`sublime`中选中变量，它出现的地方就会高亮）：出现次数、位置，在确定不会有影响的情况下合并代码。
4. C 中的数组，在汇编代码中的表示，注意寻址模式的使用，`0x8(%rbx)`可能表示的是 `array[i]`，而 `%rbx`可能表示的是 `&array`（简单粗暴的表示：带括号表示数组值，不带括号表示数组地址）。

## 技能

如 lab2说明：

> One of the nice side-effects of doing the lab is that you will get very good at using a debugger. This is a crucial skill that will pay big dividends the rest of your career.

学会了很多 gdb 调试技巧，对寄存器的使用、过程中的机制（传递控制，传递数据，分配和释放内存）、指针和汇编语言有了更深刻的认识。

CMU 的课程实验很有意思啊，很强，CSAPP 还要继续加油鸭~