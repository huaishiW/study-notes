#  1\. 背景
长期以来只知道 ELF 是一种广泛使用的文件格式规范，常指动态库、bin等，一直没动力深入研究。出于业务需求，我花了好些天仔细分析 RTOS Bin 的 Section 和 Symbol 。也是趁着这机会，查阅大量资料，完善了知识脉络。

但是可以预见，业务结束一段时间后，由于不常用，我不可避免会渐渐遗忘。从入门到遗忘，相信只要经历过深度学习的小伙伴都会有的痛苦挣扎。为了在将来需要时唤醒记忆，我决定通过这文章打下锚点，也希望此文能帮到更多人更快入门。

在学习 ELF 过程中，有一篇半译半著的文档给予了莫大帮助，属于深度好文，特此感谢原作者。
>   *
> 《Understanding_ELF.pdf》（中文）：https://paper.seebug.org/papers/Archive/refs/elf/Understanding_ELF.pdf
>
>   * 作者：赵凤阳
>
>   * 内容：翻译 ELF(v1.2) 通用规范，深入了解 ELF 每个结构的细节，并在最后提供一个实例协助理解。
#  2\. 基础概念
##  2.1 什么是 ELF 文件
ELF 的全称是 Executable and Linking Format，即“可执行可连接格式”，通俗来说，就是二进制程序。ELF 规定了这二进制程序的组织规范，所有以这规范组织的文件都叫 ELF 文件。ELF 文件有以下四类。

ELF 文件类型  |  示例  
---|---  
可重定位文件（relocatable file）  |  例如编译的过程文件 ".o"  
共享目标文件（shared object file）  |  例如 ".so" 库，以及使用动态链接的 bin  
可执行文件（executable file）  |  例如静态链接的文件  
core 文件  |  例如 Linux 上的 coredump 文件  

我们通过 **file** 命令可以识别出来
``` bash
# test.o : gcc -c test.c -o test.o  
# test-static-link.bin : gcc --static test.c -o test  
# test-dynamic-link.bin : gcc test.c -o test  

$ file test.o test-static-link.bin test-dynamic-link.bin  
test.o:                ELF ... relocatable, ...  
test-dynamic-link.bin: ELF ... shared object, ...  
test-static-link.bin:  ELF ... executable, ...  
```
除此之外，我们习惯上叫 ".o" 文件为 **目标文件（object file）** ，链接好的可执行文件叫 **bin文件** 。
##  2.2 ELF 文件结构概览
ELF 文件主要的用途有两个
  1. 构建程序，链接成动态库或者bin，一般是目标文件 ".o" 
  2. 运行程序，一般指链接好的 ".so" 或者 "bin" 
这个 ELF 文件用作不同用途，文件结构的解析角度就有点不一样，通俗来说，不同用途对需要哪些数据的要求不一样，例如构建（链接）时 节表头（Section Table Header）是必须的，但运行时却是可选的，例如运行需要段信息，而链接只有节信息。
![](https://mmbiz.qpic.cn/mmbiz_png/YHBSoNHqDiaF8PzDsYhwWiahR2njSIAvialJXA2l56x2F7lhcEkaibUufQshLYHOo6RJ9vbOl6kCokKbicehHUkiaaWQ/640?wx_fmt=png)
在上图中，程序头表（Program Header Table）紧跟在 ELF 文件头（File Header）之后，节头表（Section Header Table）紧跟在节信息之后，但在实际的文件中，这个顺序并不是固定的。在 ELF 文件的各个组成部分中， **只有ELF文件头的位置是固定的，其它内容的位置全都可变** 。
这里有一个潜在的概念，很少看到其他文章明确点出来。 **Section Header Table** 描述了所有 **节信息表** ，而 **Program Header Table** 其实描述的是所有的 **段信息表** 。
下一章节会介绍一些关键字段的含义。在这里引入这张图，主要为了引出两个重要的概念 **节（Seciton）** 和 **段（Segment）**
。由上图可以发现，链接视图有大量的 **节（Section）** ，而执行视图有 **段（Segment）** 。那么什么是段，什么又是节？

**补充**：[[概念补充及联想#Section Header Table（节头表）]]、[[概念补充及联想#Program Header Table（程序头表）]]、[[概念补充及联想#Section（节区）]]、[[概念补充及联想#Segment（段）]]、
[[概念补充及联想#Section与Segment的核心区别]]、[[概念补充及联想#为何要将 Section 合并为 Segment]]

##  2.3 节（Section） Vs. 段（Segment）
或许我们听说过 **bss段** ， **text（代码）段** ，或者 **data（数据）段** ，但其实我们口头交流的段更多是泛化的 **段**。为什么呢？
我们随便拿个 bin，不妨罗列下 程序头表（Program Header Table）：
``` bash
$ readelf -l /bin/ls  
      
    Elf file type is DYN (Shared object file)  
    Entry point 0x5850  
    There are 9 program headers, starting at offset 64  
      
    Program Headers:  
    ......  
      
    Section to Segment mapping:  
    ......  
```
**Program Headers** 下面罗列出了所有的段信息，详细如下图所示，共计有 9 个段。
![](attachments/Pasted%20image%2020260419133516.png)
**Section to Segment mapping** 罗列了 **各个段（Segment）包含了哪些节（Section）**
，是的，段是1个或者多个节的集合，详细如下文所示。
``` bash
     Section to Segment mapping:  
      Segment Sections...  
       00  
       01     .interp  
       02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .plt.got .text .fini .rodata .eh_frame_hdr .eh_frame  
       03     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss 
       04     .dynamic  
       05     .note.ABI-tag .note.gnu.build-id  
       06     .eh_frame_hdr  
       07  
       08     .init_array .fini_array .data.rel.ro .dynamic .got  
```
总的来说，段是没有名字的，但我们往往把包含 **text节** 的段叫做 **代码（text）段** ，把含有 **data节** 的段叫做**数据（data）段** 。一定程度上在口述时习惯也可以理解为 "段 = 节"，例如 **bss段** ， **rodata段** ，实际他们指**bss节** ， **rodata节** 。甚至有时候我们说 **text段** 就狭义的指 **text节** ，完全不用太纠结。
#  3\. 关键的文件结构和节
##  3.1 一些关键的文件结构
对一个 ELF 格式的文件，有这么几个特殊的结构我们需要关注的。
  * ELF 文件头（File Header）：位于文件最开始，包含了整个文件的结构信息，例如是ELF 幻数，是哪种 ELF 文件，程序头表、节头表的地址等。 
  * 程序头表（Program Header Table）：描述了所有段的信息 
  * 节头表（Section Header Table）：描述了所有节的信息 
###  3.1.1 文件头（File Header）
**` readelf -h <path/for/elf> ` ** 命令可以查看文件头信息，例如：
![](attachments/Pasted%20image%2020260419133807.png)
解读如下：
![](attachments/Pasted%20image%2020260419133845.png)
![](attachments/Pasted%20image%2020260419133901.png)
\[1]. 文件类型见 2.1 章节
\[2]. 段没名字，但节是有名字的，节的名字需要另外保存，对应到 .shstrtab 节

**补充**：[[概念补充及联想#magic（魔数）]]、[[概念补充及联想#展示信息中的两个 Version]]

###  3.1.2 程序头表（Program Header Table）
` readelf -l <path/for/elf> `  命令可以查看程序表头的信息。需要注意的是，".o"
文件一般是没有程序表头的。示例如下：
![](attachments/Pasted%20image%2020260419134026.png)
``` bash
Section to Segment mapping:  
  Segment Sections...  
   00  
   01     .interp  
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .plt.got .text .fini .rodata .eh_frame_hdr .eh_frame  
   03     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss  
   04     .dynamic  
   05     .note.ABI-tag .note.gnu.build-id  
   06     .eh_frame_hdr  
   07  
   08     .init_array .fini_array .data.rel.ro .dynamic .got
```
Program Headers罗列了所有段的信息，含义如下表：
![](attachments/Pasted%20image%2020260419134230.png)

**补充**：[[概念补充及联想#Type 的取值与含义]]、[[概念补充及联想#Align 字段的核心作用]]、[[概念补充及联想#关联规则]]

- **Section to Segment mapping**
展示了段包含哪些节，常关注的是代码段和数据段。下表只是个典型的例子，一个实际的更复杂的代码段可能包含更多的节。
![](attachments/Pasted%20image%2020260419134316.png)

###  3.1.3 节头表（Section Header Table）
**` readelf -S <path/for/elf> ` ** 命令可以查看节表头的信息，示例如下：
![](attachments/Pasted%20image%2020260419134344.png)

字段  |  含义  |  备注  
---|---|---  
Name  |  本节的名字  |  
Size  |  本节的大小  |  单位 Byte，如果节类型为 NOBITS，此值仍可能非0，但无意义  
Type  |  本节的类型  |  [1]  
EntSize  |  如果节是表类型，则表示表项大小  |  单位 Byte  
Address  |  如果需要映射到虚拟内存，则表示起始地址  |  单位 Byte  
Offset  |  本节第一个字节所在文件的偏移  |  单位 Byte  
Flags  |  权限等属性信息  |  
Link  |  略  |  可以查阅章节1提到的《UnderstandingELF.pdf》  
Info  |  略  |  可以查阅章节1提到的《UnderstandingELF.pdf》  
Align  |  大小对齐信息  |  

\[1]. 节有以下类型_
  * **NULL** ：本节头是一个无效的（非活动的）节头 
  * **PROGBITS** ：本节所含有的信息是由程序定义的，本节内容的格式和含义都由程序来决定。 
  * **SYMTAB** ：所有符号表调试信息，strip 可以干掉，不影响运行 
  * **STRTAB** ：本节是字符串表。 
  * **RELA** ：本节是一个重定位节。 
  * **HASH/GNU_HASH** ：本节包含一张哈希表。 
  * **DYNAMIC** ：本节包含的是动态连接信息。 
  * **NOTE** ：本节包含的信息用于以某种方式来标记本文件。 
  * **NOBITS** ：这一节的内容是空的，节并不占用实际的空间。 
  * **REL** ：本节是一个重定位节。 
  * **DYNSYM** ：动态链接符号表信息 
###  3.1.4 符号表（.symtab 节）
符号表不属于文件结构，但因为非常常用，也单独拎出来讲。
以一段简单的代码举例子：
``` c
#include <stdio.h>
int main(int argc, char **argv)
{
    printf("hello world");
    return 0;
}
```
编译出 hello 的 bin 之后，我们通过以下命令查看 .symtab 和 .dynsym
![](attachments/Pasted%20image%2020260419134516.png)
![](attachments/Pasted%20image%2020260419134536.png)
\[1]. 在".o" 中，如果 Ndx 不是 UND，则此值表示字符在所在节中的偏移量，在可执行文件和共享库文件中，则表示虚拟地址
\[2]. 任何一个符号表项的定义都与某一个“节”相联系，因为符号是为节而定义，在节中被引用
##  3.2 一些关键的节
###  3.2.1 "编译 + 节" 与 "链接 + 段"
在 《linux 目标文件(\*.o) bss,data,text,rodata,堆,栈》（https://blog.csdn.net/sunny04/article/details/40627311）
有张图非常形象的描述了 text、data、bss 段的作用，如下：
![](https://mmbiz.qpic.cn/mmbiz_jpg/YHBSoNHqDiaF8PzDsYhwWiahR2njSIAvialJxKjghG97MU05Mbibic6nRXBWs5icB3cwZscODFcERreVp7GmLsB1ulCw/640?wx_fmt=jpeg)
从上图可以看出，代码放在 text 段，已经初始化的变量放在 data 段，未初始化的变量放在 bss 段。详细下文再描述，在此章节，我们需要知道一个概念：
**所有变量、函数等都是一个符号，还有字符串、固定的数据等，在编译的时候会按功能、是否初始化等分类放到特定的节中。**
例如 .bss 节记录了所有未初始化的变量，.text 节保存了所有的二进制代码，.rodata 节保存了所有只读的数据。
**所以我们编译（不含链接）出来的 ".o" 目标文件，其实就是对单个 ".c/.cpp" 源码的符号、数据按不同节分类放好。**
这是编译，那么链接成可执行程序的时候呢？链接虽然涉及多个 ".o"，其实也就把相同的节合并，然后归类到段中放好。
![](https://mmbiz.qpic.cn/mmbiz_png/YHBSoNHqDiaF8PzDsYhwWiahR2njSIAvialicMrtHJApZsg59RmuqXMk6ibVHibzzicoMABPbkViatnA9Eell752tpmiajw/640?wx_fmt=png)
_此节也就很粗犷的描述，砍掉了所有枝叶，只是为了更好的更直观的理解主干，实际情况肯定更复杂。_
我们编译、链接的时候，其实可以主动限制某些符号、数据放到哪些节、哪些段的，也可以自己新建个节、段，这属于另一个大的课题了。大多时候我们也用不上，采用默认的即可。
下面，我们来看一些关键的节及其内容。
###  3.2.2 .text

前文一直有带出来，.text 节实际就是我们说的代码段，保存了一系列的可执行二进制指令。
.text 节的内容是会占 ROM 空间，并在运行时拷贝到 RAM。
` readelf -x .text <path/for/elf> `  可以以二进制 dump 出来，但没卵用，谁看得懂？此时可以用 `objdump -d -j .text <path/for/elf> ` 以汇编显示出来。
![](attachments/Pasted%20image%2020260419134818.png)
上面的 main 汇编仅仅是 ` printf("hello") ` 。
###  3.2.3 .bss

.bss 节主要保存了未初始化的变量，这里的未初始化起始也包括初始化为 0 的变量。通常是指用来存放程序中未初始化的全局变量和未初始化的局部静态变量。
.bss 节只在运行时，才会从内存分配空间，因此近乎不占 ROM 空间。
例如下面示例的两个变量都会保存到 bss。
``` c
int g_int1;
int g_int2 = 0;
```
对上述的两个变量，我们还是用 **` objdump -d -j .bss <path/for/elf> ` ** 看看：
![](attachments/Pasted%20image%2020260419134949.png)
###  3.2.4 .data

.data 节主要保存了已经初始化的变量，通常是指用来存放程序中已初始化的全局变量和已初始化的静态变量的一块内存区域。
既然已经初始化，在文件中肯定要记录初始化的值，因此 .data 节需要占 ROM 空间的，且相对 .rodata 节来说，.data
节的内容是可变的，因此运行前需要拷贝到内存中可写的区间。
objdump 对查看数据类型的节的确方便，我们继续用 objdump 看看。
![](attachments/Pasted%20image%2020260419135058.png)
代码 ` int g_int2 = 1 ` 对变量赋了非0的初始值，就不适合放到 .bss 段了，因此放到了 .data 段。从 dump 的结果也可以看到变量 g_int2 以及值 0x1 。
###  3.2.5 .rodata

.rodata 节的数据是会占 ROM 空间的，且（大多时候）在运行时拷贝到内存中。
.rodata 存的是只读数据，比如字符串常量，全局const变量 和 \#define定义的常量。例如： ` char *p = "123456" ` ,` 123456 ` 的字符串就存放在 rodata 节 中。还有一个有意思的例子：
```c
printf("the func is %s\n", __func__);  
```
` "the func is %s\n" ` 这个格式化打印字符串以及 ` __func__ ` 所指代的函数名，也是字符串常量，保存到 .rodata 节中的。
查看字符串类型的节，用 readelf 就方便多了，例如以下代码：
```c
int main(int argc, char **argv)  
{  
    printf("hello world");  
    printf("this func is %s", __func__);  
}  
```
执行以下命令可以打印字符串数据，如果需要看二进制数据，例如整型的值，把 ` -p ` 改为 ` -x ` 即可。
``` bash
readelf -p .rodata main             //main是它的文件名称
String dump of section '.rodata':  
  [     4]  hello world  
  [    10]  this func is %s  
  [    20]  main
```
这里有个小 Tips，相同的字符串只会保留 1份，因此下面两段代码效果一样，但保存的 .rodata 数据是不一样的。适当的优化可以节省 ROM空间。自己琢磨:)。
``` c
// 代码1  
printf("func %s: val is %d\n", __func__, val);  
printf("func %s: open failed\n", __func__);  
// 代码2  
printf("func main: val is %d\n", val);  
printf("func main: open failed\n");  
```
###  3.2.6 .symtab 和 .dynsym

.symtab 和 .dynsym 都是符号表，例如函数、变量等都是符号，甚至 .dynsym 是 .symtab 的子集，但是他们的作用不太一样。

**.symtab**，俗称的符号表，记录了所有符号，不管是自己定义的变量、函数，还是未定义需要动态库提供实现的所有符号。在 ”.o" 链接时必须存在，但链接成 bin 后就可去掉节省空间，例如 ` strip <path/for/bin> ` 。去掉符号表之后程序运行能定位到函数地址，但再也不知道这函数名，这也是为什么在程序 crash 时打印的栈里有时候只有地址，有时候有具体函数名。
**.dynsym**，动态链接才需要的符号表，即可包括对外提供调用的符号，也包括需要外面提供实现的符号。.dynsym 在 ".so" 或者动态链接的 bin
里是必须的。
.symtab 和 .dynsym 是占 ROM 空间的，.dynsym 会加载进内存，但 .symtab 不会。我们也可以通过 ` strip `
去掉符号表，然后通过 ` file ` 判断符号表是否已经 striped，例如下面的例子：
``` bash
$ gcc test.c -o test  
$ file test  
test: ELF 64-bit LSB shared object, ..., not stripped  
$ strip test  
test: ELF 64-bit LSB shared object, ..., stripped
```
#  4\. 利用工具解析 ELF

在上文的示例中频繁使用 readelf 和 objdump 来读取各种头表和节内容，除了这两个之外，还有一个 nm 工具，3者的功能非常相近。吃多嚼不烂，咱们以 readelf 为主，以 objdump 为辅讲解如何用工具解析 ELF 。

上文已有示例，本文主要做个汇总记录，方便将来查阅。字段的具体含义，可以查看章节3中具体的节解析。
- 获取 ELF 文件头
``` bash
readelf -h <path/for/elf>
```
- 获取程序头表（段表）
``` bash
readelf -l <path/for/elf>
```
- 获取节表（获取有哪些节）
``` bash
readelf -S <path/for/elf>     #S大写
```
- 获取符号集（列出函数、变量符号）
``` bash
# 获取所有符号表（含 .symtab 和 .dynsym）  s小写
readelf -s <path/for/elf>
# 获取动态符号表
readelf --dyn-syms <path/for/elf>
```
- 获取节内容
``` bash
# 打印节中的字符串，常用于含字符串类型的节，例如 .rodata 节
readelf -p <name/of/section> <path/for/elf>
# 以二进制打印节，常用于非字符串类型的节，例如 .bss，.data 节
readelf -x <name/of/section> <path/for/elf>
# 以汇编打印二进制代码
objdump -d -j <name/of/section> <path/for/elf>
```
#  5\. ELF 在磁盘 Vs. ELF 加载到内存

在 《完全剖析 - Linux虚拟内存空间管理》（https://cloud.tencent.com/developer/article/1835295）
一文有张图画出了虚拟内存的空间分布情况，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YHBSoNHqDiaF8PzDsYhwWiahR2njSIAvialvMKajdMZpxK3TtBfeL2CGoqTicv34mkvxe0yoBTcbpxxxWJvcQhrZPw/640?wx_fmt=png)

我们关注最底下3个区间，其实跟 ELF 的内容是能对应上的。用一张新图来表示两者的关系：

![](https://mmbiz.qpic.cn/mmbiz_png/YHBSoNHqDiaF8PzDsYhwWiahR2njSIAvialU2fuaZvkzias3ia9WYWlaDsORicfxvDkma3JUe8PaZJEiawSvj8ngO5ZQQ/640?wx_fmt=png)

可执行文件的 代码段、数据段等会拷贝到内存中，BSS 段虽然没数据，但也记录了有哪些变量，会拷贝到内存可写区域，而动态库是 map 到 mmap 区的。

