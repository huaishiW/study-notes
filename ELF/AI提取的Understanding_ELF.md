`内容不完整简易阅读PDF原文件`
# 深入理解ELF格式

**实战详解**

> *"让你陷入麻烦的不是你不知道的事，而是你确信不疑的错事。"*
> — 马克·吐温

本书是2016年发布的同名博客系列的更新和扩展版本。原始博客旨在提供ELF格式的实用概述，多年来已成为学习ELF最受欢迎的参考资料之一。原始系列已重写、更新、扩展并自费出版，以便更多人受益。本书从解释ELF是什么开始，概述格式，然后以实用方式涵盖格式的所有主要方面。如果你曾想进入逆向工程和二进制漏洞利用领域，但被明显缺乏适合初学者的资源所阻碍，那么这本书就是为你准备的。

**关键词：** ELF、二进制、逆向工程、漏洞利用

---

## 第一章：简介

### 什么是ELF？

可执行和可链接格式（Executable and Linkable Format，ELF）是Unix类系统（如Linux和macOS）使用的标准二进制格式。它用于可执行文件、目标代码、共享库和核心转储。它在System V应用程序二进制接口（ABI）中定义，Linux实现则在Linux Foundation的文档中有描述。

在我们开始之前，让我们快速了解不同类型的ELF文件。ELF文件可以是以下类型之一：

| 类型 | 描述 |
|------|------|
| **可重定位文件** | 包含可与其他目标文件链接以生成可执行文件或共享对象文件的数据和代码 |
| **可执行文件** | 包含可执行的程序 |
| **共享对象文件** | 包含可在程序链接阶段与其他目标文件链接的代码和数据，或在执行期间加载到可执行文件中的代码和数据 |
| **核心转储文件** | 包含崩溃程序的内存快照 |

在本书的上下文中，我们将研究共享对象文件和可执行文件，因为这是你遇到的最常见的ELF文件类型。

### 格式概述

ELF文件由三个主要组件组成：

| 组件 | 描述 |
|------|------|
| **ELF头（ELF header）** | 包含关于文件的元数据，如文件类型、架构、入口点、程序头表偏移量、节区头表偏移量等 |
| **程序头表（Program header table）** | 包含如何将文件加载到内存的信息，如内存段、它们的权限和对齐方式 |
| **节区头表（Section header table）** | 包含关于文件中节区的信息，如节区名称、节区类型、节区标志、节区地址、节区偏移量、节区大小等 |

下图显示了ELF文件的高层结构：

```
+------------------+
| ELF头            |
+------------------+
| 节区头表          | <-- 指向
|                  |     节区
+------------------+
|                  |
| 节区             |
|                  |
+------------------+

+------------------+
| ELF头            |
+------------------+
| 程序头表          | <-- 指向
|                  |     段
+------------------+
|                  |
| 段              |
|                  |
+------------------+
```

如图所示，节区头表指向节区，程序头表指向段。重要的是要注意，节区是链接器工作的较小不可分割单元，而段是程序加载器工作的较大单元。段由一个或多个节区组成。下图显示了节区和段之间的关系：

```
+---------+  +---------+  +---------+
| 节区   |  | 节区   |  | 节区   |
+---------+  +---------+  +---------+
      ^           ^           ^
      |           |           |
      +-----+-----+-----+-----+
            |
      +-----v-----+
      |   段     |
      +-----------+
```

现在让我们深入研究这三个主要组件，从ELF头开始。

---

## 第二章：解析ELF头

当我们查看ELF文件时，ELF头是我们看到的第一个东西。它位于文件的开头，包含关于文件的元数据。让我们先看看ELF头的结构。

### ELF头的位置和大小

ELF头位于文件偏移量`0x00`处，其大小是固定的。对于32位ELF文件，ELF头是52字节；对于64位ELF文件，ELF头是64字节。

### ELF头的结构

ELF头包含以下字段：

| 字段 | 描述 |
|------|------|
| `e_ident` | 魔数和其他信息 |
| `e_type` | 对象文件类型 |
| `e_machine` | 架构 |
| `e_version` | 对象文件版本 |
| `e_entry` | 入口点虚拟地址 |
| `e_phoff` | 程序头表的文件偏移量 |
| `e_shoff` | 节区头表的文件偏移量 |
| `e_flags` | 处理器特定标志 |
| `e_ehsize` | ELF头大小 |
| `e_phentsize` | 程序头表项大小 |
| `e_phnum` | 程序头表项数量 |
| `e_shentsize` | 节区头表项大小 |
| `e_shnum` | 节区头表项数量 |
| `e_shstrndx` | 节区头字符串表索引 |

现在我们将仔细查看这些字段中的每一个。

### `e_ident`字段

`e_ident`字段是一个16字节的数组，包含魔数和其他信息。下表描述了该字段的内容：

| 索引 | 名称 | 描述 |
|------|------|------|
| 0 | `EI_MAG0` | 魔数字节0，必须为`0x7F` |
| 1 | `EI_MAG1` | 魔数字节1，必须为`'E'` |
| 2 | `EI_MAG2` | 魔数字节2，必须为`'L'` |
| 3 | `EI_MAG3` | 魔数字节3，必须为`'F'` |
| 4 | `EI_CLASS` | 文件类别 |
| 5 | `EI_DATA` | 数据编码 |
| 6 | `EI_VERSION` | 文件版本 |
| 7 | `EI_OSABI` | OS ABI标识 |
| 8 | `EI_ABIVERSION` | ABI版本 |
| 9-15 | `EI_PAD` | 填充 |

前四个字节`EI_MAG0`到`EI_MAG3`必须是`0x7F`、`'E'`、`'L'`、`'F'`。这是标识文件为ELF文件的魔数。如果你尝试读取非ELF文件，你将看不到这些魔数。

`EI_CLASS`字节指示文件类别，即字长。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `ELFCLASSNONE` | 0 | 无效类别 |
| `ELFCLASS32` | 1 | 32位对象 |
| `ELFCLASS64` | 2 | 64位对象 |

`EI_DATA`字节指示数据编码，即文件的字节序。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `ELFDATANONE` | 0 | 无效数据编码 |
| `ELFDATA2LSB` | 1 | 小端序 |
| `ELFDATA2MSB` | 2 | 大端序 |

`EI_VERSION`字节指示文件版本。唯一有效的值是`EV_CURRENT`，即`1`。

`EI_OSABI`字节指示OS ABI标识。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `ELFOSABI_NONE` | 0 | UNIX System V ABI |
| `ELFOSABI_HPUX` | 1 | HP-UX ABI |
| `ELFOSABI_NETBSD` | 2 | NetBSD ABI |
| `ELFOSABI_GNU` | 3 | GNU ABI |
| `ELFOSABI_HURD` | 4 | GNU/Hurd ABI |
| `ELFOSABI_SOLARIS` | 6 | Solaris ABI |
| `ELFOSABI_AIX` | 7 | AIX ABI |
| `ELFOSABI_IRIX` | 8 | IRIX ABI |
| `ELFOSABI_FREEBSD` | 9 | FreeBSD ABI |
| `ELFOSABI_TRU64` | 10 | TRU64 UNIX ABI |
| `ELFOSABI_MODESTO` | 11 | Modesto ABI |
| `ELFOSABI_OPENBSD` | 12 | OpenBSD ABI |
| `ELFOSABI_OPENVMS` | 13 | OpenVMS ABI |
| `ELFOSABI_NSK` | 14 | HP Non-Stop Kernel ABI |
| `ELFOSABI_AROS` | 15 | AROS ABI |
| `ELFOSABI_FENIXOS` | 16 | FenixOS ABI |
| `ELFOSABI_CLOUDABI` | 17 | CloudABI ABI |
| `ELFOSABI_32BIT` | 18 | 32位环境ABI |
| `ELFOSABI_ARM` | 97 | ARM架构ABI |
| `ELFOSABI_STANDALONE` | 255 | 独立ABI |

对于Linux，`EI_OSABI`字节通常设置为`ELFOSABI_NONE`、`ELFOSABI_GNU`或`ELFOSABI_ARM`。对于macOS，通常设置为`ELFOSABI_NONE`。对于FreeBSD，通常设置为`ELFOSABI_FREEBSD`。

### `e_type`字段

`e_type`字段指示对象文件类型。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `ET_NONE` | 0 | 无文件类型 |
| `ET_REL` | 1 | 可重定位文件 |
| `ET_EXEC` | 2 | 可执行文件 |
| `ET_DYN` | 3 | 共享对象文件 |
| `ET_CORE` | 4 | 核心文件 |

### `e_machine`字段

`e_machine`字段指示架构。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `EM_NONE` | 0 | 无机器 |
| `EM_386` | 3 | Intel 80386 |
| `EM_X86_64` | 62 | AMD x86-64 |
| `EM_ARM` | 40 | ARM |
| `EM_AARCH64` | 183 | ARM AArch64 |
| `EM_RISCV` | 243 | RISC-V |

### `e_version`字段

`e_version`字段指示对象文件版本。唯一有效的值是`EV_CURRENT`，即`1`。

### `e_entry`字段

`e_entry`字段指示入口点虚拟地址。这是程序开始执行的地址。

### 其他字段

| 字段 | 描述 |
|------|------|
| `e_phoff` | 程序头表的文件偏移量 |
| `e_shoff` | 节区头表的文件偏移量 |
| `e_flags` | 处理器特定标志 |
| `e_ehsize` | ELF头大小（字节） |
| `e_phentsize` | 程序头表项大小（字节） |
| `e_phnum` | 程序头表项数量 |
| `e_shentsize` | 节区头表项大小（字节） |
| `e_shnum` | 节区头表项数量 |
| `e_shstrndx` | 节区头字符串表索引 |

---

## 第三章：深入节区

在上一章中，我们看了ELF头，它包含关于文件的元数据。在本章中，我们将了解节区，它们是链接器工作的较小不可分割单元。

### 什么是节区？

节区是链接器工作的较小不可分割单元。它们包含程序的实际数据和代码，如代码、数据、只读数据、调试信息等。

### 节区头表

节区头表是节区头条目的数组。每个节区头条目包含关于节区的信息，如节区名称、节区类型、节区标志、节区地址、节区偏移量、节区大小等。

### 节区头条目结构

节区头条目包含以下字段：

| 字段 | 描述 |
|------|------|
| `sh_name` | 节区名称（字符串表索引） |
| `sh_type` | 节区类型 |
| `sh_flags` | 节区标志 |
| `sh_addr` | 节区虚拟地址 |
| `sh_offset` | 节区文件偏移量 |
| `sh_size` | 节区大小（字节） |
| `sh_link` | 链接到另一个节区 |
| `sh_info` | 附加信息 |
| `sh_addralign` | 节区对齐 |
| `sh_entsize` | 条目大小（如果节区包含表） |

### 节区类型

`sh_type`字段指示节区类型。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `SHT_NULL` | 0 | 非活动节区头 |
| `SHT_PROGBITS` | 1 | 程序定义的数据 |
| `SHT_SYMTAB` | 2 | 符号表 |
| `SHT_STRTAB` | 3 | 字符串表 |
| `SHT_RELA` | 4 | 带加数的重定位条目 |
| `SHT_HASH` | 5 | 符号哈希表 |
| `SHT_DYNAMIC` | 6 | 动态链接信息 |
| `SHT_NOTE` | 7 | 注释 |
| `SHT_NOBITS` | 8 | 无文件映像（BSS） |
| `SHT_REL` | 9 | 不带加数的重定位条目 |
| `SHT_SHLIB` | 10 | 保留 |
| `SHT_DYNSYM` | 11 | 动态链接器符号表 |
| `SHT_INIT_ARRAY` | 14 | 初始化函数 |
| `SHT_FINI_ARRAY` | 15 | 终止函数 |
| `SHT_PREINIT_ARRAY` | 16 | 预初始化函数 |
| `SHT_GROUP` | 17 | 节区组 |
| `SHT_SYMTAB_SHNDX` | 18 | 扩展节区索引 |

### 节区标志

`sh_flags`字段指示节区标志。定义了以下值：

| 名称 | 值 | 描述 |
|------|-----|------|
| `SHF_WRITE` | 0x1 | 可写节区 |
| `SHF_ALLOC` | 0x2 | 执行期间占用内存 |
| `SHF_EXECINSTR` | 0x4 | 可执行节区 |
| `SHF_MERGE` | 0x10 | 节区数据可合并 |
| `SHF_STRINGS` | 0x20 | 节区数据包含以null结尾的字符串 |
| `SHF_INFO_LINK` | 0x40 | `sh_info`包含节区头表索引 |
| `SHF_LINK_ORDER` | 0x80 | 特殊排序要求 |
| `SHF_OS_NONCONFORMING` | 0x100 | 需要OS特定处理 |
| `SHF_GROUP` | 0x200 | 节区组成员 |
| `SHF_TLS` | 0x400 | 线程局部存储 |
| `SHF_COMPRESSED` | 0x800 | 节区数据已压缩 |

### 常见节区

| 节区 | 类型 | 标志 | 描述 |
|------|------|------|------|
| `.text` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_EXECINSTR` | 程序代码 |
| `.data` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_WRITE` | 已初始化数据 |
| `.bss` | `SHT_NOBITS` | `SHF_ALLOC` \| `SHF_WRITE` | 未初始化数据 |
| `.rodata` | `SHT_PROGBITS` | `SHF_ALLOC` | 只读数据 |
| `.symtab` | `SHT_SYMTAB` | 无 | 符号表 |
| `.strtab` | `SHT_STRTAB` | 无 | 字符串表 |
| `.shstrtab` | `SHT_STRTAB` | 无 | 节区名称字符串表 |
| `.dynsym` | `SHT_DYNSYM` | `SHF_ALLOC` | 动态符号表 |
| `.dynstr` | `SHT_STRTAB` | `SHF_ALLOC` | 动态字符串表 |
| `.rela.dyn` | `SHT_RELA` | `SHF_ALLOC` | 动态重定位 |
| `.rela.plt` | `SHT_RELA` | `SHF_ALLOC` | PLT重定位 |
| `.init` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_EXECINSTR` | 初始化代码 |
| `.fini` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_EXECINSTR` | 终止代码 |
| `.init_array` | `SHT_INIT_ARRAY` | `SHF_ALLOC` \| `SHF_WRITE` | 初始化函数 |
| `.fini_array` | `SHT_FINI_ARRAY` | `SHF_ALLOC` \| `SHF_WRITE` | 终止函数 |
| `.dynamic` | `SHT_DYNAMIC` | `SHF_ALLOC` \| `SHF_WRITE` | 动态链接信息 |
| `.plt` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_EXECINSTR` | 程序链接表 |
| `.got` | `SHT_PROGBITS` | `SHF_ALLOC` \| `SHF_WRITE` | 全局偏移表 |
| `.eh_frame` | `SHT_PROGBITS` | `SHF_ALLOC` | 异常处理帧 |

---

## 第四章：重定位

在上一章中，我们看了节区，它是链接器工作的较小不可分割单元。在本章中，我们将了解重定位，这是链接器用来调整节区中的代码和数据的机制。

### 什么是重定位？

重定位是链接器用来调整节区中的代码和数据的机制。这是必要的，因为链接器在创建输出文件时不知道符号的最终地址。因此，它在代码和数据中留下占位符，当程序加载到内存时需要用符号的最终地址填充这些占位符。

### 重定位条目结构

对于32位ELF文件，`SHT_RELA`重定位条目的结构如下：

| 字段 | 描述 |
|------|------|
| `r_offset` | 引用的地址 |
| `r_info` | 符号索引和重定位类型 |
| `r_addend` | 加数 |

对于64位ELF文件，`SHT_RELA`重定位条目的结构如下：

| 字段 | 描述 |
|------|------|
| `r_offset` | 引用的地址 |
| `r_info` | 符号索引和重定位类型 |
| `r_addend` | 加数 |

`r_info`字段是一个32位（对于32位ELF）或64位（对于64位）值，包含符号索引和重定位类型。符号索引存储在高24位，重定位类型存储在低8位（对于32位ELF）或低32位（对于64位ELF）。

### x86-64常见重定位类型

| 名称 | 描述 |
|------|------|
| `R_X86_64_NONE` | 无重定位 |
| `R_X86_64_64` | 直接64位 |
| `R_X86_64_PC32` | PC相对32位有符号 |
| `R_X86_64_GOT32` | 32位GOT条目 |
| `R_X86_64_PLT32` | 32位PLT条目 |
| `R_X86_64_COPY` | 运行时复制符号 |
| `R_X86_64_GLOB_DAT` | 创建GOT条目 |
| `R_X86_64_JUMP_SLOT` | 创建PLT条目 |
| `R_X86_64_RELATIVE` | 按程序基址调整 |
| `R_X86_64_GOTPCREL` | 到GOT的32位有符号PC相对偏移 |

### 重定位工作原理

当链接器创建输出文件时，它查看重定位条目并相应地调整节区中的代码和数据。链接器用符号的最终地址填充占位符。

当程序加载到内存时，动态链接器查看重定位条目并相应地调整节区中的代码和数据。这样做是为了确保程序无论加载到内存的哪里都能正确运行。

对于某些重定位类型，调整在加载时由动态链接器完成。对于其他重定位类型，调整在运行时由动态链接器在首次引用符号时完成。这称为延迟绑定。

---

## 第五章：符号

在之前的章节中，我们看了节区和重定位。在本章中，我们将了解符号，这是链接器和程序用来引用函数和变量的机制。

### 什么是符号？

符号是链接器和程序用来引用函数和变量的机制。链接器使用它们来解析目标文件之间的引用，程序使用它们在运行时引用函数和变量。

### 符号表条目结构

对于64位ELF文件，符号条目的结构如下：

| 字段 | 描述 |
|------|------|
| `st_name` | 符号名称（字符串表索引） |
| `st_info` | 符号类型和绑定 |
| `st_other` | 符号可见性 |
| `st_shndx` | 节区头索引 |
| `st_value` | 符号值 |
| `st_size` | 符号大小 |

### 符号类型

| 名称 | 值 | 描述 |
|------|-----|------|
| `STT_NOTYPE` | 0 | 无类型 |
| `STT_OBJECT` | 1 | 数据对象 |
| `STT_FUNC` | 2 | 函数 |
| `STT_SECTION` | 3 | 节区 |
| `STT_FILE` | 4 | 文件名 |
| `STT_COMMON` | 5 | 通用数据对象 |
| `STT_TLS` | 6 | 线程局部数据对象 |
| `STT_GNU_IFUNC` | 10 | 间接函数 |

### 符号绑定

| 名称 | 值 | 描述 |
|------|-----|------|
| `STB_LOCAL` | 0 | 局部符号 |
| `STB_GLOBAL` | 1 | 全局符号 |
| `STB_WEAK` | 2 | 弱符号 |
| `STB_GNU_UNIQUE` | 10 | 唯一符号（GNU扩展） |

### 符号可见性

| 名称 | 值 | 描述 |
|------|-----|------|
| `STV_DEFAULT` | 0 | 默认符号可见性 |
| `STV_INTERNAL` | 1 | 内部符号可见性 |
| `STV_HIDDEN` | 2 | 隐藏符号可见性 |
| `STV_PROTECTED` | 3 | 受保护符号可见性 |

### 特殊节区索引

| 名称 | 值 | 描述 |
|------|-----|------|
| `SHN_UNDEF` | 0 | 未定义符号 |
| `SHN_ABS` | 0xFFF1 | 绝对符号 |
| `SHN_COMMON` | 0xFFF2 | 通用符号 |

### GNU IFUNC

GNU IFUNC（间接函数）是一种允许函数根据系统能力在运行时解析的机制。函数返回要使用的实际实现的指针。

---

## 第六章：动态链接

在之前的章节中，我们看了节区、重定位和符号。在本章中，我们将了解动态链接，这是一种允许程序在运行时使用共享对象的机制。

### 什么是动态链接？

动态链接是允许程序在运行时使用共享对象的机制。它被称为"动态"，因为链接是在运行时完成的，而不是在编译时。

### 动态段

动态段是包含关于动态链接过程信息的节区。它是一个`ElfNN_Dyn`结构表，其中`NN`是字长（32或64）。

### 常见动态条目类型

| 名称 | 描述 |
|------|------|
| `DT_NULL` | 动态数组结束 |
| `DT_NEEDED` | 需要库的名称 |
| `DT_HASH` | 符号哈希表地址 |
| `DT_STRTAB` | 字符串表地址 |
| `DT_SYMTAB` | 符号表地址 |
| `DT_RELA` | 重定位表地址 |
| `DT_RELASZ` | 重定位表大小 |
| `DT_RELAENT` | 重定位条目大小 |
| `DT_STRSZ` | 字符串表大小 |
| `DT_SYMENT` | 符号表条目大小 |
| `DT_INIT` | 初始化函数地址 |
| `DT_FINI` | 终止函数地址 |
| `DT_SONAME` | 共享对象名称 |
| `DT_RPATH` | 库搜索路径（已废弃） |
| `DT_RUNPATH` | 库搜索路径 |
| `DT_GNU_HASH` | GNU哈希表地址 |
| `DT_JMPREL` | PLT重定位表地址 |
| `DT_BIND_NOW` | 立即绑定 |
| `DT_TEXTREL` | 代码段中的重定位 |

### 程序解释器

程序解释器（也称为动态链接器或ld.so）是一个负责将程序及其依赖加载到内存并链接在一起的共享对象。

### 动态链接工作原理

当程序加载到内存时，动态链接器将程序及其依赖加载到内存并链接在一起。动态链接器使用动态段和重定位条目来完成此操作。

首先，动态链接器查看动态段中的`DT_NEEDED`条目，以找到程序依赖的共享对象的名称。然后，它将每个共享对象加载到内存并解析程序引用的符号。

动态链接器使用符号表（`.dynsym`）和哈希表（`.hash`和`.gnu.hash`）来解析符号。一旦所有符号都被解析，动态链接器就将重定位应用到程序及其依赖。

最后，动态链接器调用初始化函数（`.init_array`）并将控制权转移到程序。

### 延迟绑定

延迟绑定是一种将符号解析延迟到首次引用的机制。这样做是为了加快程序的加载。

当程序引用在共享对象中定义的符号时，动态链接器拦截该引用并在此时解析符号。第一次引用符号时，动态链接器解析它并更新GOT（全局偏移表），以便后续引用直接转到符号。

### 全局偏移表（GOT）

全局偏移表（GOT）是一个用于解析符号的地址表。它位于`.got`节区中。

### 程序链接表（PLT）

程序链接表（PLT）是一个用于调用共享对象中的函数的代码序列表。它位于`.plt`节区中。

---

## 第七章：程序头表

在本章中，我们将了解程序头表，这是程序加载器用来将程序加载到内存的机制。

### 程序头条目结构

对于64位ELF文件，程序头条目的结构如下：

| 字段 | 描述 |
|------|------|
| `p_type` | 段类型 |
| `p_flags` | 段标志 |
| `p_offset` | 段文件偏移量 |
| `p_vaddr` | 段虚拟地址 |
| `p_paddr` | 段物理地址 |
| `p_filesz` | 文件中段的大小 |
| `p_memsz` | 内存中段的大小 |
| `p_align` | 段对齐 |

### 常见段类型

| 名称 | 值 | 描述 |
|------|-----|------|
| `PT_NULL` | 0 | 未使用的程序头 |
| `PT_LOAD` | 1 | 可加载段 |
| `PT_DYNAMIC` | 2 | 动态链接信息 |
| `PT_INTERP` | 3 | 程序解释器 |
| `PT_NOTE` | 4 | 注释信息 |
| `PT_PHDR` | 6 | 程序头表本身 |
| `PT_TLS` | 7 | 线程局部存储 |
| `PT_GNU_EH_FRAME` | 0x6474e550 | GNU EH帧 |
| `PT_GNU_STACK` | 0x6474e551 | GNU栈 |
| `PT_GNU_RELRO` | 0x6474e552 | GNU relro |
| `PT_GNU_PROPERTY` | 0x6474e553 | GNU属性 |

### 段标志

| 名称 | 值 | 描述 |
|------|-----|------|
| `PF_X` | 0x1 | 可执行段 |
| `PF_W` | 0x2 | 可写段 |
| `PF_R` | 0x4 | 可读段 |

---

## 第八章：加载和执行程序

### 程序如何加载

当你运行程序时，shell调用`execve`系统调用，由内核处理。内核查看程序头表并将每个可加载段加载到内存。然后，它设置栈并将控制权转移到程序。

### 动态链接程序如何加载

当你运行动态链接的程序时，shell调用`execve`系统调用，由内核处理。内核查看程序头表并将程序加载到内存。然后，它查看`PT_INTERP`段以找到程序解释器（ld.so）并将程序解释器加载到内存。最后，它将控制权转移到程序解释器。

程序解释器（ld.so）查看动态段并将程序依赖的每个共享对象加载到内存。然后，它解析符号并应用重定位。最后，它将控制权转移到程序。

### 栈的设置

当内核将控制权转移到程序时，栈设置如下：

- 参数计数（`argc`）
- 参数字符串（`argv`）
- 环境变量（`envp`）
- 辅助向量（`auxv`）

辅助向量是提供附加信息的键值对表。一些常见的辅助向量条目：

| 名称 | 描述 |
|------|------|
| `AT_NULL` | 辅助向量结束 |
| `AT_PHDR` | 程序头表地址 |
| `AT_PHENT` | 程序头条目大小 |
| `AT_PHNUM` | 程序头条目数量 |
| `AT_PAGESZ` | 系统页大小 |
| `AT_BASE` | 解释器基地址 |
| `AT_ENTRY` | 程序入口点地址 |
| `AT_UID` | 真实用户ID |
| `AT_EUID` | 有效用户ID |
| `AT_GID` | 真实组ID |
| `AT_EGID` | 有效组ID |
| `AT_RANDOM` | 随机字节地址 |
| `AT_HWCAP` | 硬件能力 |
| `AT_PLATFORM` | 平台字符串 |

---

## 第九章：ABI（应用程序二进制接口）

### 什么是ABI？

ABI（应用程序二进制接口）是一组定义程序如何在二进制级别与系统交互的规则。它包括调用约定、寄存器使用、内存布局、对象文件格式等。

### System V ABI

System V ABI是Unix类系统（如Linux和macOS）使用的标准ABI。它定义了调用约定、寄存器使用、内存布局、对象文件格式等。

### Linux ABI

Linux ABI基于System V ABI，并添加了Linux特定的扩展。

### Windows ABI

Windows ABI与System V ABI不同。它使用不同的调用约定、不同的对象文件格式（PE）等。

---

## 第十章：工具

### readelf

`readelf`是用于显示ELF文件信息的工具。它可以显示ELF头、程序头表、节区头表、符号、重定位、动态段等。

```bash
# 查看ELF头
readelf -h elf_file

# 查看程序头
readelf -l elf_file

# 查看节区
readelf -S elf_file

# 查看符号
readelf -s elf_file

# 查看动态段
readelf -d elf_file
```

### objdump

`objdump`是用于显示目标文件和可执行文件信息的工具。它可以显示ELF头、程序头表、节区头表、符号、反汇编等。

```bash
# 反汇编
objdump -d elf_file

# 查看节区内容
objdump -s elf_file

# 查看文件头
objdump -f elf_file
```

### nm

`nm`是用于显示目标文件或可执行文件中的符号的工具。

```bash
# 查看所有符号
nm elf_file

# 查看全局符号
nm -g elf_file

# 符号解码
nm -C elf_file
```

### ldd

`ldd`是用于显示可执行文件或共享对象的共享对象依赖项的工具。

```bash
# 查看依赖
ldd elf_file
```

### objcopy

`objcopy`是用于复制和转换目标文件的工具。

### strip

`strip`是用于从目标文件或可执行文件中移除符号和其他信息的工具。

### hexdump

`hexdump`是用于以十六进制格式显示文件内容的工具。

---

## 第十一章：Linux内核中的ELF

### Linux内核如何处理ELF文件

当Linux内核收到`execve`系统调用时，它查看文件以确定它是否是ELF文件。它通过检查ELF头中的魔数来做到这一点。如果魔数是`0x7F`、`'E'`、`'L'`、`'F'`，则内核知道该文件是ELF文件。

然后，内核查看程序头表并将每个可加载段加载到内存。最后，它设置栈并将控制权转移到程序。

### binfmt_elf模块

Linux内核使用`binfmt_elf`模块来处理ELF文件。该模块负责检查魔数、加载段和设置栈。

---

## 第十二章：总结

在本书中，我们研究了ELF格式的各个组件。我们涵盖了ELF头、程序头表、节区头表、节区、重定位、符号、动态链接、程序加载器、ABI、工具以及Linux内核对ELF文件的处理。

希望本书能让你对ELF格式有一个很好的理解，并帮助你开始进行逆向工程和二进制漏洞利用。

---

## 附录A：术语表

| 术语 | 描述 |
|------|------|
| ABI | 应用程序二进制接口（Application Binary Interface） |
| BSS | 符号起始块（未初始化数据） |
| GOT | 全局偏移表（Global Offset Table） |
| PLT | 程序链接表（Procedure Linkage Table） |
| ELF | 可执行和可链接格式（Executable and Linkable Format） |
| PIC | 位置无关代码（Position Independent Code） |
| PIE | 位置无关可执行文件（Position Independent Executable） |
| TLS | 线程局部存储（Thread Local Storage） |

---

## 附录B：参考文献

1. [System V应用程序二进制接口](https://refspecs.linuxfoundation.org/elf/elf.pdf)
2. [Linux Foundation文档](https://refspecs.linuxfoundation.org/)
3. [Intel x86-64 ABI](https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.99.pdf)
4. [DWARF标准](https://dwarfstd.org/)
5. [ELF: From the Ground Up](http://www.mse-music-production.org/wp-content/uploads/2015/12/elf-from-the-ground-up.pdf)

---

*本书由AI技术生成。*
