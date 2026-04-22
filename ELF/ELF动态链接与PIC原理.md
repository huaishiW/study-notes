# ELF 动态链接与 PIC 原理

> 本文档系统整理了动态链接器 (ld-linux-x86-64.so.2)、符号表 (.symtab)、字符串表 (.strtab)、重定位表 (.rela.dyn / .rela.plt)、PIC (Position Independent Code) 的核心原理，以及它们在 x86_64 Linux 环境下协同工作的机制。

---

## 1. 动态链接器 (ld-linux-x86-64.so.2)

### 1.1 概述

动态链接器是 Linux 系统中负责**在程序运行时完成符号解析与重定位**的特殊共享库。它的路径通常为 `/lib64/ld-linux-x86-64.so.2`（x86_64 架构）。

当内核加载一个带有 `PT_INTERP` 程序头的 ELF 可执行文件时，内核并不直接把控制权交给可执行文件，而是先加载并运行动态链接器，由它完成所有共享库的映射和符号解析后，才最终跳转到程序的入口点。

- **它**本身就是一个共享库（Shared Object），但它极其特殊。在Linux系统中，当你运行一个动态链接的执行文件时，内核（Kernel）做的第一件事不是跳转到你程序的`main`函数，而是先加载这个`ld-linux`到内存中，并将控制权改变它。

- **它的职责：** 1.检查你的程序依赖了哪些`.so`文件（比如`libc.so.6`）。 2.把这些`.so`文件映射（mmap）到当前进程的虚拟内存空间中。 3.**解析符号并执行重定位**（然后填好所有外部函数和变量的真实内存地址）。 4.一切准备就绪后，把控制权交还给你程序的入口点（`_start`）

### 1.2 动态链接器的工作流程

1. **加载阶段**：内核根据 `PT_INTERP` 段找到解释器路径，将其映射进进程地址空间，移交控制权。
2. **初始化**：解析被加载可执行文件的 `.dynamic` 段，获取 `DT_SYMTAB`（动态符号表）、`DT_STRTAB`（动态字符串表）、`DT_RELA`（重定位表）、`DT_JMPREL`（PLT 重定位表）、`DT_PLTGOT`（GOT 地址）等关键信息。
3. **加载共享库**：遍历 `DT_NEEDED` 条目，递归加载所有依赖的共享库，构建"链接映射"（link map）链表。
4. **应用重定位**：遍历 `.rela.dyn` 中的条目，在启动时（eager relocation）完成所有非 PLT 函数的地址解析；如果使用 lazy binding，则 `.rela.plt` 中的 PLT 重定位留到首次调用时才解析。
5. **符号解析**：对每个需要解析的符号，在动态符号表（`.dynsym`）中查找，并按库的加载顺序（link-map 链表）处理符号覆盖（interposition）。
6. **保护 RELRO 区域**：完成重定位后，对标记为 RELRO 的区域调用 `mprotect` 设为只读，防止运行时被篡改。
7. **执行初始化函数**：调用 `DT_INIT` 和 `DT_INIT_ARRAY` 中登记的构造函数，最后跳转到程序入口点。

### 1.3 Lazy Binding（延迟绑定）

默认情况下，动态链接器采用**懒绑定**策略：只有当某个 PLT 函数第一次被调用时，才通过 PLT 桩跳转到动态链接器的解析函数（`_dl_runtime_resolve`），完成符号解析并将结果写入对应的 GOT 条目。后续调用直接通过已填充的 GOT 条目跳转到目标函数，无需再次解析。

可通过以下方式禁用懒绑定（启用 eager binding）：
- 链接时添加 `-z now` 参数
- 设置 `DT_BIND_NOW` 动态标签

---

## 2. 符号表 (.symtab / .dynsym)

### 2.1 两套符号表的区别

| 表格 | 用途 | 链接器使用 | 运行时使用 |
|------|------|-----------|-----------|
| `.symtab` + `.strtab` | 编译/链接时的完整符号信息（包含调试符号） | 是 | **否** |
| `.dynsym` + `.dynstr` | 动态链接专用的导出/导入符号 | 是 | **是** |

`.dynsym` 是 `.symtab` 的子集，只包含需要跨共享库边界解析的符号。动态链接器在运行时只使用 `.dynsym` / `.dynstr`。

### 2.2 Elf64_Sym 结构体（24 字节）

| 字段 | 类型 | 大小 | 说明 |
|------|------|------|------|
| `st_name` | Elf64_Word | 4 B | 到字符串表的字节偏移量（指向 `.dynstr` 或 `.strtab`） |
| `st_info` | unsigned char | 1 B | 符号类型与绑定属性（见下） |
| `st_other` | unsigned char | 1 B | 符号可见性（通常为 0） |
| `st_shndx` | Elf64_Half | 2 B | 所在节的下标（SHN_ABS / SHN_UNDEF 等特殊值） |
| `st_value` | Elf64_Addr | 8 B | 符号的地址（加载后为绝对地址，未加载为相对偏移） |
| `st_size` | Elf64_Xword | 8 B | 符号对象的大小（字节数） |

### 2.3 st_info 的编码

- **低 4 位**（`ELF64_ST_TYPE`）：符号类型
  - `STT_NOTYPE` (0) — 无类型（未定义）
  - `STT_OBJECT` (1) — 数据对象（变量、数组等）
  - `STT_FUNC` (2) — 函数
  - `STT_SECTION` (3) — 段（节）符号
  - `STT_FILE` (4) — 文件名符号
  - `STT_COMMON` (5) — 通用块（C/C++ 的 Tentative 定义）
  - `STT_TLS` (6) — 线程局部存储
  - `STT_IFUNC` (10) — 间接函数（需要调用解析器）

- **高 4 位**（`ELF64_ST_BIND`）：符号绑定
  - `STB_LOCAL` (0) — 局部（不可见）
  - `STB_GLOBAL` (1) — 全局（可跨对象解析）
  - `STB_WEAK` (2) — 弱绑定（与全局类似但可被覆盖）

### 2.4 示例

```c
// test.c
int global_var = 42;
void my_func() { }
```

用 `readelf -s /lib64/ld-linux-x86-64.so.2 | head -30` 可以看到动态符号表中的条目，每个 `st_name` 都是一个到 `.dynstr` 的偏移量。

---

## 3. 字符串表 (.strtab / .dynstr)

### 3.1 原理

字符串表以**空字符分隔**的变长字符串序列存储在 ELF 的 `.strtab`（静态）和 `.dynstr`（动态）节中。符号名、节名等都以**偏移量**的形式引用字符串表，而非直接内嵌在符号结构体中。

例如，字符串表从偏移 0 开始：

```
偏移:  0  1  2  3  4  5  6  7  8  9  ...
内容: \0 g  l  o  b  a  l  _  v  a  r  \0  m  y  _  f  u  n  c  \0  ...
```

符号 `st_name = 1` 指向 "lobal_var"，`st_name = 13` 指向 "my_func"。

### 3.2 动态字符串表 .dynstr

- `.dynstr` 必须是连续的、以 `\0` 结尾的字符串序列。
- 动态链接器通过 `DT_STRTAB`（地址）和 `DT_STRSZ`（大小）找到它。
- 所有动态符号（`.dynsym` 中的条目）只能引用 `.dynstr`，不能引用 `.strtab`。

### 3.3 符号名称解析过程

```
r_info >> 32  →  符号在 .dynsym 中的下标
↓                          ↓
.st_name 偏移    →    在 .dynstr 中查找字符串   →   符号名字符串
```

- **字符串表 ( `.strtab`)：这是一个巨大的字节存储，里面塞满了以空字符`\0`结尾的字符串。比如`"printf\0main\0my_var\0"`。它不包含任何结构，只是用来存储名称。**
- **符号表 ( `.symtab`)：这是一个结构体备份（比如`Elf64_Sym`）。每一个入口代表一个符号（函数、全局变量等）。它记录了符号的值（地址或偏移量）、大小、类型（是函数还是数据），**最重要的是，它包含一个指向`.strtab`的索引**，告诉链接器：“我的名字在字符串表的第 X 个字节处”。

**关键区别（动态链接特供）：** 静态链接时主要用`.symtab`和`.strtab`。但在最终生成的动态执行文件或`.so`文件中，为了节省内存，还会存在`.dynsym`（动态符号表）**和**`.dynstr`（动态字符串表）**。它们是`.symtab`专用版本，只保留动态链接器在运行时真正需要的那些“导出”和“导入”的符号。

---

## 4. 重定位表 (.rela.dyn / .rela.plt)

重定位（Relocation）就是“修改地址”的过程。因为共享库每次加载到内存的地址可能不太一样，所以代码里引用外部函数或变量的绝对地址都是假的，需要在运行时。这些表就是指导动态修改链接器如何修改地址的说明书

### 4.1 Elf64_Rela 结构体（24 字节）

| 字段 | 类型 | 说明 |
|------|------|------|
| `r_offset` | Elf64_Addr (8 B) | **待重定位的位置**（需修改的内存地址或 GOT 条目地址） |
| `r_info` | Elf64_Xword (8 B) | 高 32 位 = 符号表下标；低 32 位 = 重定位类型 |
| `r_addend` | Elf64_Sxword (8 B) | **加数**（参与计算的值，x86_64 统一使用 RELA 格式） |

`r_addend` 是 RELA 与 REL 的核心区别：RELA 显式存储加数，REL 则从被修改位置读取。

### 4.2 .rela.dyn vs .rela.plt

| 特性   | `.rela.dyn`                                             | `.rela.plt`                       |
| ---- | ------------------------------------------------------- | --------------------------------- |
| 动态标签 | `DT_RELA` / `DT_RELASZ`                                 | `DT_JMPREL`                       |
| 内容   | 全局数据引用、非 PLT 代码引用的重定位                                   | PLT 函数调用的重定位（JUMP_SLOT 类型）        |
| 应用时机 | **启动时必须完成**（否则程序无法安全运行）                                 | 默认懒绑定（首次调用时）；`DT_BIND_NOW` 时启动时完成 |
| 典型类型 | `R_X86_64_GLOB_DAT`, `R_X86_64_64`, `R_X86_64_RELATIVE` | `R_X86_64_JUMP_SLOT`              |

- **`.rela.dyn`（数据重定位）：**主要用于**数据变量**的重定位。当你引用的全局变量来自另一个共享库时，它的地址记录在全局偏移表（GOT，Global Offset Table）中。`.rela.dyn`指示动态链接器在程序启动时，去替换那些外部变量的真实地址，并填入GOT表的相应槽位中。

- **`.rela.plt`（函数重定位）：**主要用于**外部函数**的重定位。现代系统为了加快程序的启动速度，对函数采用了**延迟绑定（Lazy Binding）**机制。`.rela.plt`指示修改过程链接表（PLT，Procedure Linkage Table）和GOT。只有当你的程序**第一次**真正调用某个外部函数（比如`printf`）时，动态链接器才会去解析它的真实地址并记录下来，后续调用就直接跳转了。

### 4.3 x86_64 常用重定位类型

| 类型名 | 值 | 计算公式 | 说明 |
|--------|----|---------|------|
| `R_X86_64_64` | 1 | `S + A` | 64 位绝对地址（数据） |
| `R_X86_64_PC32` | 2 | `S + A - P` | 32 位 PC 相对（代码） |
| `R_X86_64_GOT32` | 3 | `G + A` | GOT 条目的 32 位相对地址 |
| `R_X86_64_PLT32` | 4 | `L + A - P` | PLT 条目的 32 位 PC 相对 |
| `R_X86_64_COPY` | 5 | — | 将共享库数据复制到可执行文件 BSS |
| `R_X86_64_GLOB_DAT` | 6 | `S` | 将 GOT 条目设为符号绝对地址（无偏移） |
| `R_X86_64_JUMP_SLOT` | 7 | `S` | PLT 的 GOT 条目，lazy binding 写入目标地址 |
| `R_X86_64_RELATIVE` | 8 | `B + A` | 基于加载基址的重定位（无需符号） |
| `R_X86_64_GOTPCREL` | 9 | `G + GOT + A - P` | PC 相对方式访问 GOT |
| `R_X86_64_IRELATIVE` | 37 | `B + A`（调用解析器） | IFUNC 间接函数，由解析器返回最终地址 |

> 公式中：`S` = 符号的绝对地址；`A` = `r_addend`；`P` = 重定位位置（`r_offset`）的地址；`B` = 加载基址；`G` = GOT 条目地址；`L` = PLT 条目地址。

### 4.4 GOT 与 PLT 协同工作原理

**Global Offset Table (GOT)**：位于`.got.plt` 节（PLT 专用 GOT）和 `.got` 节（数据 GOT）。每个外部数据引用和每个外部函数调用都有一个对应的 GOT 条目，初始状态保存的是动态链接器的解析函数地址（懒绑定时）或目标地址（eager binding 时）。

**Procedure Linkage Table (PLT)**：位于 `.plt` 节。每个需要懒绑定的外部函数对应一个 PLT 桩（stub），典型的 PLT 桩如下：

```asm
# PLT[0] — 所有 PLT 桩的入口（通用解析跳转）
push   GOT[1]          # 压入 link_map 标识
jmp    GOT[2]          # 跳转到 _dl_runtime_resolve

# PLT[n] — 具体函数的桩（n >= 1）
push   index           # 压入重定位表下标
jmp    PLT[0]          # 跳转回 PLT[0] 统一入口
```

**工作流程（懒绑定）**：

```
调用 my_func@plt
  → PLT[n]桩: push  relocation_index
            jmp  PLT[0]
  → PLT[0]:   push  link_map
            jmp  GOT[2] (_dl_runtime_resolve)
  → ld.so 解析符号，写 S 到 GOT[PLT+n]
  → 跳转到 S（目标函数）

后续调用:
  → PLT[n]:  jmp  GOT[PLT+n]  (已填充，直接跳转)
```

---

## 5. PIC (Position Independent Code) 原理

### 5.1 为什么需要 PIC

共享库（`.so`）在加载时**地址不固定**——内核可能在每次加载时分配不同的虚拟地址（ASLR 启用时更是如此）。如果代码中使用绝对地址，直接引用全局变量或函数将无法工作。

PIC 的核心思想：**代码不依赖自身加载到的绝对地址**，而是通过**相对地址**或**间接寻址**来访问导出的符号，使得同一份机器码在任意地址都能正确运行。

### 5.2 RIP 相对寻址（x86_64 PIC 的关键）

x86_64 架构提供了一种以 `RIP`（指令指针）为基准的寻址方式：

```asm
mov  rax, [rip + offset]   ; RIP 是当前指令地址，offset 是编译时已知的常量
```

这条指令执行时，CPU 自动计算 `RIP + offset` 作为最终地址。由于 `RIP` 会在运行时被自动更新为当前指令的地址，**offset 只需编码到指令中**，无需知道代码的绝对加载地址。

这使得以下 PIC 访问成为可能：

- **访问全局数据**：通过 `[rip + offset_to_GOT_entry]` 找到 GOT 条目，再从 GOT 条目读取数据变量的绝对地址。
- **调用函数**：通过 `[rip + offset_to_GOT_entry]` 找到 PLT/GOT 中的函数地址并跳转。

### 5.3 PIC 代码中的 GOT 访问

在 PIC 共享库中，编译器将**每个全局变量和函数引用**编译成对 GOT 的间接访问：

```c
// PIC 共享库中的代码
extern int global_var;

int foo() {
    return global_var;  // 编译为: mov eax, [rip + offset_to_GOT_global_var]
}
```

`[rip + offset]` 指向 GOT 中对应的条目，GOT 条目在运行时被动态链接器填充为全局变量的真实地址。

### 5.4 PIC vs 非 PIC（对比）

| 特性 | 非 PIC（静态链接/可执行文件） | PIC（共享库 / PIE） |
|------|---------------------------|-------------------|
| 代码是否依赖加载地址 | 是（使用绝对地址） | 否（使用 RIP 相对或 GOT 间接） |
| 能否随机化加载地址 | 否（需固定基址） | 是（ASLR 可应用于 PIC） |
| 对数据的引用 | `mov rax, [sym_addr]` | `mov rax, [rip + GOT_off]` → GOT |
| 对函数的调用 | `call sym_addr`（直接） | `call [rip + GOT_off]` 或 `call plt_sym` |
| 产生的重定位类型 | `R_X86_64_64`, `R_X86_64_PC32` | `R_X86_64_GOTPCREL`, `R_X86_64_JUMP_SLOT` 等 |
| 可被多个进程共享 | 否（文本段包含绝对地址不可共享） | 是（相同代码可在不同进程中映射到不同地址） |

### 5.5 PIE（位置无关可执行文件）

`gcc -fPIE -pie` 编译出的**可执行文件**（而非共享库）也采用 PIC 机制。其行为与 PIC 共享库类似，但最终由动态链接器在加载时确定其加载基址 `B`，并应用所有 `R_X86_64_RELATIVE` 等重定位。

非 PIE 可执行文件通常被链接到固定地址（如 x86_64 Linux 上传统上使用 `0x400000`），不需要在运行时重定位自身代码。

### 5.6 PIC 中的相对重定位 R_X86_64_RELATIVE

对于 PIC 代码，大多数重定位是 `R_X86_64_RELATIVE` 类型：

```
计算公式：B + A
其中 B = 加载基址，A = r_addend
```

这类重定位只需要知道加载基址 `B`，**不需要进行符号查找**，因此是最高效的一类重定位，也是 PIC/PIE 加载速度快的关键原因。

---

## 6. 综合运行时协作示例

以下是一个完整的懒绑定调用链：

```
程序调用 printf@plt
  ↓
PLT[printf] 桩:
  push  reloc_index      ; 0x3（示例）
  jmp   PLT[0]
  ↓
PLT[0]:
  push  link_map
  jmp   GOT[2]           ; → _dl_runtime_resolve
  ↓
ld-linux 解析:
  1. 读取 reloc_index，查 .rela.plt
  2. r_info 高32位 = 符号下标 → 查 .dynsym → st_name → .dynstr 查 "printf"
  3. 沿 link_map 链表搜索 printf 的定义
  4. 获得 printf 的实际加载地址 S
  5. 将 S 写入 GOT[printf_plt_entry]  (r_offset 位置)
  6. 跳转到 S 执行 printf
  ↓
后续 printf 调用:
  PLT[printf]:
    jmp  GOT[printf_plt_entry]  ; 直接跳转，不再经过解析器
```

最核心也是最精彩的机制就是**延迟绑定（Lazy Binding）**以及配合它工作的**PLT（过程链接表）**和**GOT（全局偏移表）**。

这是动态链接的“魔法”所在。前面提到，共享库的代码是位置无关的（PIC），它在运行时才确定外部函数的真实地址。但是，一个庞大的程序可能依赖了几千个外部函数（比如`printf`，等）。如果程序刚启动时，动态链接器把这几千`malloc`个`strlen`函数的真实地址全部查好，程序启动会非常慢。

为了解决这个问题，Linux采用了**延迟绑定**：**只有当你第一次真正调用这个函数时，动态链接器才会去查找它的真实地址。**

为了实现这个机制，编译器设计了两个表：**GOT**和**PLT**。

### 两个核心角色的分工

- **GOT（Global Offset Table，全局偏移表）**：
  - **位置**：在数据段（`.data`或`.bss`附近），**可写**。
  - **本质上**：一个卸载托盘。每个需要调用的外部函数这里都有一个对应的”槽位”（比如`GOT[printf]`）。
  - **状态**：它最终应该保存`printf`在内存中的真实绝对地址。但在第一次调用之前，它保存的是一个”假地址”。

- **PLT（Procedure Linkage Table，过程链接表）**：
  - **位置**：在代码段（`.text`附近），**只读**。
  - **本质上**：有很多很短的自定义代码片段（跳板）。每个外部函数都有一个对应的PLT边界（比如`printf@plt`）。你的用户代码里凡是调用`printf`的地方，实际上都是`call printf@plt`。

### 魔法的发生：第一次调用 vs 第二次调用

当你第一次调用`printf`时，会发生极其精彩的”左脚踩右脚”的操作：

**第一次调用（查找并绑定）**

1. **用户代码**：调用`call printf@plt`。
2. **PLT跳板**：进入`printf@plt`代码。它的第一条指令是去查GOT表（`jmp *GOT[printf]`无条件跳转到GOT表里存的那个地址）。
3. **GOT骗局**：此时是第一次运行，`GOT[printf]`里面存的**不是** `printf`真实地址，而是**直接指向了PLT中下一条指令的地址**！
4. **返回 PLT**：指令执行流又神奇地弹回了 PLT。接下来的 PLT 指令会做两件事：
   - 把`printf`这个符号放在重定位表中的编号（ID）压入栈中（告诉系统”我想找谁”）。
   - 跳转到公共`PLT[0]`。
5. **召唤神龙**：`PLT[0]`的作用是跳转到**动态链接器(ld-linux.so)**的解析函数（`_dl_runtime_resolve`）。
6. **动态解析**：动态链接器根据刚才压入栈的ID，去符号表里找到`printf`的真实内存地址。
7. **修改GOT**：找到真实地址后，动态链接器把这个真实地址覆盖掉刚才`GOT[printf]`里的”假地址”。
8. **执行目标**：动态链接器将控制权限重置的真正`printf`函数。

**第二次调用（高速通道）**

1. **用户代码**：再次调用`call printf@plt`。
2. **PLT跳板**：进入`printf@plt`，执行`jmp *GOT[printf]`。
3. **直接命中**：因为在第一次调用时，`GOT[printf]`已经被动态链接器填入了真实的`printf`地址，所以这次跳转就直接飞到了libc库里的`printf`真正代码处！
    

再往后，无论调用多少次，都是直接通过GOT表飞向目标，不再需要动态链接器的介入了。

---

## 7. RELRO 安全机制

**Partial RELRO** (`-z relro`)：
- 部分节（`.data`, `.bss`, `.got`）设为只读
- `.rela.plt`（JUMP_SLOT）仍可写，支持懒绑定

**Full RELRO** (`-z relro -z now`)：
- 所有 PLT 重定位在**启动时完成**
- `.got.plt` 整体设为只读
- 彻底消除 GOT 覆写攻击面

---

## 8. 常用检查命令

```bash
# 查看程序解释器
readelf -l prog | grep INTERP

# 查看动态段（关键标签）
readelf -d prog

# 查看符号表（动态符号）
readelf -s prog        # .dynsym
readelf -s -s prog    # .symtab（两套同时显示）

# 查看所有节
readelf -S prog

# 查看重定位表
readelf -r prog
objdump -R prog       # 更易读

# 查看 PLT 桩（反汇编）
objdump -d -j .plt prog

# 查看 GOT 内容（需要调试或 gdb）
objdump -s -j .got.plt prog
```

---

## 9. 参考资料

- [ELF-64 Object File Format - uClibc](https://uclibc.org/docs/elf-64-gen.pdf)
- [elf(5) - Linux manual page](https://man7.org/linux/man-pages/man5/elf.5.html)
- [System V ABI AMD64 - Oracle](https://docs.oracle.com/cd/E19683-01/816-1386/chapter6-54839/index.html)
- [Relocations - Some Assembly Required](https://mindfruit.co.uk/posts/2012/06/relocations/)
- [System V ABI - AMD64 (PDF)](https://people.freebsd.org/~obrien/amd64-elf-abi.pdf)
- [Relative relocations and RELR - MaskRay](https://maskray.me/blog/2021-10-31-relative-relocations-and-relr)
- [All about Procedure Linkage Table - MaskRay](https://maskray.me/blog/2021-09-19-all-about-procedure-linkage-table)
- [A look at dynamic linking - LWN.net](https://lwn.net/Articles/961117/)
- [What Is /lib64/ld-linux-x86-64.so.2? - Baeldung](https://www.baeldung.com/linux/dynamic-linker)
- [Hardening ELF binaries using RELRO - Red Hat](https://www.redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro)
- [Dynamic Linking in ELF - Kevin Koo](https://kevinkoo001.github.io/blog/2016/dynamic-linking-in-elf/)
- [glibc elf/dl-reloc.c](https://codebrowser.dev/glibc/glibc/elf/dl-reloc.c.html)
- [System V ABI x86_64-0.95.pdf](https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.95.pdf)
