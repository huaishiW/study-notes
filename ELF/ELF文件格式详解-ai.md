# ELF 文件格式详解

## 目录

1. [概述](#1-概述)
2. [ELF文件结构](#2-elf文件结构)
3. [ELF头部 (ELF Header)](#3-elf头部-elf-header)
4. [程序头部 (Program Header)](#4-程序头部-program-header)
5. [节区头部 (Section Header)](#5-节区头部-section-header)
6. [节区类型](#6-节区类型)
7. [符号表 (Symbol Table)](#7-符号表-symbol-table)
8. [字符串表 (String Table)](#8-字符串表-string-table)
9. [重定位表 (Relocation Table)](#9-重定位表-relocation-table)
10. [动态链接](#10-动态链接)
11. [线程局部存储 (TLS)](#11-线程局部存储-tls)
12. [安全机制](#12-安全机制)
13. [DWARF调试信息](#13-dwarf调试信息)
14. [压缩节区](#14-压缩节区)
15. [符号版本控制](#15-符号版本控制)
16. [平台特定扩展](#16-平台特定扩展)
17. [实例分析](#17-实例分析)
18. [工具与命令](#18-工具与命令)
19. [参考文献](#19-参考文献)

---

## 1. 概述

### 1.1 什么是ELF

ELF (Executable and Linkable Format) 是一种通用的二进制文件格式标准，由UNIX系统实验室 (USL) 作为应用程序二进制接口 (ABI) 的一部分开发。它是一种跨平台、可扩展的格式，用于：
- 可执行文件 (Executable)
- 目标文件 (Object File)
- 共享对象 (Shared Object)
- 核心转储文件 (Core Dump)

### 1.2 ELF的历史

- **1983年**: 首次在Unix System V中引入
- **1995年**: 成为Linux标准
- **1999年**: 被正式承认为 Unix 和 Linux 的标准二进制文件格式
- **现在**: 广泛用于大多数现代Unix-like系统 (Linux, FreeBSD, Solaris等)

### 1.3 ELF的设计目标

1. **可扩展性**: 支持多种处理器架构
2. **跨平台性**: 支持多种操作系统
3. **灵活性**: 支持静态和动态链接
4. **高效性**: 支持执行时内存映射

### 1.4 ELF版本历史

| 年份 | 版本/特性 | 说明 |
|------|----------|------|
| 1983 | 原始ELF | Unix System V 首次引入 |
| 1995 | ELF v1 | Linux标准采用 |
| 1999 | ELF v2 | 成为Unix/Linux标准二进制格式 |
| 2003 | TLS v1 | 线程局部存储支持加入ELF规范 |
| 2006 | GCC 4.x | GNU_PROPERTY扩展，RELRO改进 |
| 2010 | ELF v3 | 压缩节区(SHF_COMPRESSED)标准 |
| 2015 | GNU_HASH v2 | 更快的符号查找 |
| 2018 | RISC-V支持 | EM_RISCV正式加入 |
| 2020 | ARM64普及 | AArch64成为主流移动架构 |
| 2022 | ELF 4.2 | gABI更新，支持超过65000个节区 |
| 2023 | CET支持 | 控制流完整性(IBT/SHSTK) |
| 2024 | ELF 4.3 | 最新规范草案，支持SHT_RELR、紧凑节区头 |
| 2025 | zstd调试压缩 | LLVM 15+/binutils 2.39+ 全面支持 |
| 2025 | CREL提案 | 紧凑重定位格式讨论中 |

### 1.5 ELF规范发展

**最新ELF规范**: gABI (generic ABI) 4.3版本正在开发中(Cary Coutant主导)，主要特性包括：

- **SHT_RELR**: 新的标准相对重定位节区类型
- **紧凑节区头表格式**: 减少大型二进制文件节区头开销
- **SHT_SYMTAB_SHNDX扩展**: 支持更大符号表
- **新的处理器架构支持**: 持续更新EM_*值
- **安全特性标准化**: GNU_PROPERTY扩展正式纳入

**相关链接**:
- [ELF 4.3规范草案 (GitHub)](https://github.com/hjl-tools/linux-abi/tree/master/doc)
- [gABI规范](https://gabi.xinuos.com)
- [Linux Foundation ELF规范](https://refspecs.linuxfoundation.org/)
- [MaskRay博客 - ELF进化](https://maskray.me/blog/2024-05-26-evolution-of-elf-object-file-format)

---

## 2. ELF文件结构

### 2.1 整体布局

```
+-------------------+
|   ELF Header      |  <- 固定位置，通常在文件开头
+-------------------+
|   Program Header  |  <- 可选，仅可执行文件和共享库有
|      Table        |
+-------------------+
|   Section 1       |
+-------------------+
|   Section 2       |
+-------------------+
|       ...         |
+-------------------+
|   Section N       |
+-------------------+
|   Section Header  |  <- 可执行文件和共享库有
|      Table        |
+-------------------+
```

### 2.2 文件类型

ELF文件有三种类型，由ELF头的 `e_type` 字段标识：

| 类型值 | 名称 | 描述 |
|--------|------|------|
| ET_NONE | 0 | 未知类型 |
| ET_REL  | 1 | 可重定位文件 (Relocatable file) - 目标文件 |
| ET_EXEC | 2 | 可执行文件 (Executable file) |
| ET_DYN  | 3 | 共享对象文件 (Shared object file) - 动态库 |
| ET_CORE | 4 | 核心转储文件 (Core dump file) |

### 2.3 数据编码

ELF支持两种数据编码方式：

| 枚举值 | 名称 | 描述 |
|--------|------|------|
| ELFDATA2LSB | 1 | 小端序 (Least Significant Byte first) |
| ELFDATA2MSB | 2 | 大端序 (Most Significant Byte first) |

### 2.4 处理器架构

ELF支持多种处理器架构，部分枚举值：

| 值 | 名称 | 描述 |
|----|------|------|
| EM_NONE | 0 | 无架构 |
| EM_386 | 3 | Intel 80386 |
| EM_X86_64 | 62 | AMD x86-64 |
| EM_ARM | 40 | ARM (32位) |
| EM_AARCH64 | 183 | ARM AArch64 (64位) |
| EM_RISCV | 243 | RISC-V (64位为主) |
| EM_SPARC | 2 | SPARC |
| EM_MIPS | 8 | MIPS (32/64位) |
| EM_PowerPC | 20 | PowerPC (32位) |
| EM_PPC64 | 21 | PowerPC 64位 |
| EM_IA64 | 50 | Intel Itanium |
| EM_LOONGARCH | 258 | LoongArch (龙芯) |
| EM_ARC | 93 | ARC (Synopsys) |
| EM_NDS32 | 167 | Andes NDS32 |
| EM_OPENRISC | 185 | OpenRISC |
| EM_RISC-V | 243 | RISC-V |

---

## 3. ELF头部 (ELF Header)

### 3.1 32位与64位ELF头

ELF同时存在32位和64位两种版本，结构略有不同。

### 3.2 ELF64头结构

```c
#define EI_NIDENT 16

typedef struct {
    unsigned char e_ident[EI_NIDENT]; /* 魔数和其他信息 */
    Elf64_Half    e_type;            /* 文件类型 */
    Elf64_Half    e_machine;         /* 目标架构 */
    Elf64_Word    e_version;         /* 文件版本 */
    Elf64_Addr    e_entry;           /* 程序入口点虚拟地址 */
    Elf64_Off     e_phoff;           /* 程序头部表偏移 */
    Elf64_Off     e_shoff;           /* 节区头部表偏移 */
    Elf64_Word    e_flags;           /* 处理器特定标志 */
    Elf64_Half    e_ehsize;          /* ELF头大小 */
    Elf64_Half    e_phentsize;       /* 程序头部表项大小 */
    Elf64_Half    e_phnum;           /* 程序头部表项数量 */
    Elf64_Half    e_shentsize;       /* 节区头部表项大小 */
    Elf64_Half    e_shnum;           /* 节区头部表项数量 */
    Elf64_Half    e_shstrndx;        /* 节区名字符串表索引 */
} Elf64_Ehdr;
```

### 3.3 e_ident (魔数)

`e_ident[EI_NIDENT]` 数组包含16字节，定义如下：

| 索引 | 名称 | 描述 |
|------|------|------|
| 0-3 | EI_MAG0-3 | 魔数: 0x7F, 'E', 'L', 'F' |
| 4 | EI_CLASS | 文件类别: 1=32位, 2=64位 |
| 5 | EI_DATA | 数据编码: 1=小端, 2=大端 |
| 6 | EI_VERSION | ELF版本号: 通常为EV_CURRENT (1) |
| 7 | EI_OSABI | OS ABI标识 |
| 8-15 | EI_ABIVERSION | ABI版本 |
| 8-15 | EI_PAD | 填充字节 (未使用时) |

### 3.4 魔数验证

ELF文件的魔数固定为 `\x7FELF`：
```
$ xxd -l 16 /bin/ls
00000000: 7f45 4c46 0202 0101 0000 0000 0000 0000  .ELF............
```

### 3.5 使用readelf查看ELF头

```bash
$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 02 01 01 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, big endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Entry point address:               0x0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          13512 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 28
```

---

## 4. 程序头部 (Program Header)

### 4.1 作用

程序头部描述了如何将文件映射到内存，是运行时加载器 (loader) 使用的结构。

### 4.2 Elf64_Phdr 结构

```c
typedef struct {
    Elf64_Word    p_type;    /* 段类型 */
    Elf64_Word    p_flags;   /* 段标志 */
    Elf64_Off     p_offset;  /* 文件中段的偏移 */
    Elf64_Addr    p_vaddr;    /* 内存中的虚拟地址 */
    Elf64_Addr    p_paddr;    /* 物理地址 (通常忽略) */
    Elf64_Xword   p_filesz;  /* 文件中段的大小 */
    Elf64_Xword   p_memsz;   /* 内存中段的大小 */
    Elf64_Xword   p_align;   /* 对齐 */
} Elf64_Phdr;
```

### 4.3 段类型 (p_type)

| 类型值 | 名称 | 描述 |
|--------|------|------|
| PT_NULL | 0 | 未使用，忽略 |
| PT_LOAD | 1 | 可加载段 |
| PT_DYNAMIC | 2 | 动态链接信息 |
| PT_INTERP | 3 | 解释器路径 |
| PT_NOTE | 4 | 附加信息 |
| PT_SHLIB | 5 | 保留 |
| PT_PHDR | 6 | 程序头部表自身 |
| PT_TLS | 7 | 线程局部存储 |
| PT_GNU_EH_FRAME | 0x6474e550 | GCC异常处理帧 |
| PT_GNU_STACK | 0x6474e551 | 堆栈标志 |
| PT_GNU_RELRO | 0x6474e552 | 只读重定位 |
| PT_GNU_PROPERTY | 0x6474e553 | GNU属性 |

### 4.4 段标志 (p_flags)

| 标志值 | 名称 | 描述 |
|--------|------|------|
| PF_X | 1 | 可执行 |
| PF_W | 2 | 可写 |
| PF_R | 4 | 可读 |
| PF_MASKOS | 0x0ff00000 | OS特定 |
| PF_MASKPROC | 0xf0000000 | 处理器特定 |

### 4.5 PT_LOAD 详解

`PT_LOAD` 是最重要的段类型，描述了可执行代码和数据如何加载到内存：

```
p_filesz < p_memsz: .bss节区额外内存为零
p_filesz == p_memsz: 没有.bss节区
```

### 4.6 查看程序头部

```bash
$ readelf -l /bin/ls

Elf file type is DYN (Shared object file)
Entry point 0x0
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x13a868 0x13a868 R E 0x200000
  LOAD           0x13b000 0x000000000013b000 0x000000000013b000 0x023a70 0x02e4e0 RW  0x200000
  DYNAMIC        0x15eb48 0x000000000015eb48 0x000000000015eb48 0x0001f0 0x0001f0 RW  0x8
  NOTE           0x000268 0x0000000000000268 0x0000000000000268 0x000020 0x000020 R   0x8
  NOTE           0x000288 0x0000000000000288 0x0000000000000288 0x000024 0x000024 R   0x4
  GNU_EH_FRAME   0x13a7ec 0x000000000013a7ec 0x000000000013a7ec 0x000574 0x000574 R   0x4
  GNU_STACK       0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x13b000 0x000000000013b000 0x000000000013b000 0x023a70 0x023a70 R   0x1
```

---

## 5. 节区头部 (Section Header)

### 5.1 作用

节区头部描述了文件的各个节区 (sections)，主要用于链接器和调试器。

### 5.2 Elf64_Shdr 结构

```c
typedef struct {
    Elf64_Word    sh_name;      /* 节区名称 (字符串表索引) */
    Elf64_Word    sh_type;      /* 节区类型 */
    Elf64_Xword   sh_flags;     /* 节区标志 */
    Elf64_Addr    sh_addr;      /* 虚拟地址 (无则为0) */
    Elf64_Off     sh_offset;    /* 文件中偏移 */
    Elf64_Xword   sh_size;      /* 节区大小 */
    Elf64_Word    sh_link;      /* 链接信息 */
    Elf64_Word    sh_info;      /* 附加信息 */
    Elf64_Xword   sh_addralign; /* 对齐 */
    Elf64_Xword   sh_entsize;   /* 表项大小 (如果是表) */
} Elf64_Shdr;
```

### 5.3 节区类型 (sh_type)

| 类型值 | 名称 | 描述 |
|--------|------|------|
| SHT_NULL | 0 | 未使用 |
| SHT_PROGBITS | 1 | 程序定义信息 |
| SHT_SYMTAB | 2 | 符号表 |
| SHT_STRTAB | 3 | 字符串表 |
| SHT_RELA | 4 | 重定位表 (带加数) |
| SHT_HASH | 5 | 符号哈希表 |
| SHT_DYNAMIC | 6 | 动态链接信息 |
| SHT_NOTE | 7 | 注释信息 |
| SHT_NOBITS | 8 | 不占用文件空间的节区 (.bss) |
| SHT_REL | 9 | 重定位表 (不带加数) |
| SHT_SHLIB | 10 | 保留 |
| SHT_DYNSYM | 11 | 动态链接符号表 |
| SHT_INIT_ARRAY | 14 | 初始化函数数组 |
| SHT_FINI_ARRAY | 15 | 结束函数数组 |
| SHT_PREINIT_ARRAY | 16 | 预初始化函数数组 |
| SHT_GROUP | 17 | 节区组 |
| SHT_SYMTAB_SHNDX | 18 | 扩展节区索引 |
| SHT_RELR | 19 | RELR相对重定位表 (ELF 4.3) |
| SHT_ANDROID_REL | 0x60000000 | Android REL (已废弃) |
| SHT_ANDROID_RELA | 0x60000001 | Android RELA (已废弃) |
| SHT_LLVM_ODRTAB | 0x6fff4c00 | LLVM ODRTAB |
| SHT_GNU_ATTRIBUTES | 0x6ffffff5 | GNU属性 |
| SHT_GNU_HASH | 0x6ffffff6 | GNU风格哈希表 |
| SHT_GNU_LIBLIST | 0x6ffffff7 | 库列表 |
| SHT_GNU_verdef | 0x6ffffffd | 版本定义 |
| SHT_GNU_verneed | 0x6ffffffe | 版本需求 |

### 5.3.1 紧凑节区头表 (Compact Section Header Table)

ELF 4.3规范引入了紧凑节区头表格式，旨在减少大型二进制文件的节区头开销：

**传统格式问题**:
- 每个Elf64_Shdr结构固定64字节
- 大型项目可能有数万个节区
- 节区头表可能占用数MB空间

**紧凑格式**:
- 使用压缩编码减少平均大小
- 对齐信息从结构中移出，单独存储
- 目标：减少30-50%的节区头空间

**状态**:
- ELF 4.3规范草案阶段
- 主要链接器正在评估实现

### 5.4 节区标志 (sh_flags)

| 标志值 | 名称 | 描述 |
|--------|------|------|
| SHF_WRITE | 1 | 可写 |
| SHF_ALLOC | 2 | 占用内存 |
| SHF_EXECINSTR | 4 | 可执行指令 |
| SHF_MERGE | 16 | 可合并 |
| SHF_STRINGS | 32 | 包含字符串 |
| SHF_INFO_LINK | 64 | sh_info包含索引 |
| SHF_LINK_ORDER | 128 | 维持链接顺序 |
| SHF_OS_NONCONFORMING | 256 | OS特定 |
| SHF_GROUP | 512 | 节区组成员 |
| SHF_TLS | 1024 | 线程局部存储 |
| SHF_COMPRESSED | 0x800 | 压缩节区 |

### 5.5 查看节区头部

```bash
$ readelf -S /bin/ls
There are 31 section headers, starting at offset 0x34b8:

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00     0   0  0
  [ 1] .note.gnu.build-id NOTE            0000000000000270 000270 000024 00   A  0   0  4
  [ 2] .gnu.hash         GNU_HASH        0000000000000298 000298 00347c 00   A  3   0  8
  [ 3] .dynsym           DYNSYM          0000000000003618 003618 0011a8 18   A  4   1  8
  [ 4] .dynstr           STRTAB          00000000000047c0 0047c0 00e206 00   A  0   0  1
  [ 5] .gnu.version      VERSYM          0000000000125a 00  5   a2   02   A  3   0  2
  [ 6] .gnu.version_r    VERNEED          000000000012b0 006b0 0000d0 00   A  4   2  8
  [ 7] .rela.dyn          RELA            000000000012880 006880 003a98 18   A  3   0  8
  [ 8] .rela.plt          RELA            000000000016318 006318 003618 18   A  3  13  8
  [ 9] .init             PROGBITS        000000000001900 01900 000017 00  AX  0   0  4
  [10] .plt              PROGBITS        000000000001920 01920 000250 00  AX  0   0 16
  [11] .text             PROGBITS        000000000001c80 001c80 00b942 00  AX  0   0 16
  [12] .fini             PROGBITS        00000000000d4d0 00d4d0 00000d 00  AX  0   0  4
  [13] .rodata           PROGBITS        00000000000d500 00d500 02a400 00   A  0   0 32
  [14] .data             PROGBITS        000000000013b000 13b000 01a00 00  WA  0   0  8
  [15] .bss              NOBITS          000000000013ca00 13ca00 01c000 00  WA  0   0  8
  [16] .tdata            PROGBITS        000000000013ca00 13ca00 000010 00  WAT  0   0  1
  [17] .tbss             NOBITS          000000000013ca10 13ca10 000008 00 WAT  0   0  1
  [18] .got.plt          PROGBITS        000000000013f000 13f000 000400 08  WA  0   0  8
  [19] .data.rel.ro      PROGBITS        000000000014a000 14a000 00e000 00  WA  0   0 32
  [20] .dynamic          DYNAMIC         000000000015eb48 15eb48 0001f0 10  WA  3   0  8
  [21] .got              PROGBITS        000000000015ed40 15ed40 0000c0 08  WA  0   0  8
  [22] .symtab           SYMTAB          000000000016010 16010 00a8d8 24  23  25  8
  [23] .strtab           STRTAB          0000000000208e8 208e8 00afb4 00      0   0  1
  [24] .shstrtab         STRTAB          00000000002189c 2189c 0011c8 00      0   0  1
  [25] .text.startup    PROGBITS        00000000000d4e0 00d4e0 000070 00  AX  0   0  1
  [26] .text.unlikely    PROGBITS        00000000000d550 00d550 0000e0 00  AX  0   0  1
  [27] .interp           PROGBITS        000000000000238 0238 00001c 00   A  0   0  1
  [28] .comment          PROGBITS        000000000000000 02354 00001d 01  MS  0   0  1
  [29] .note.GNU-stack   PROGBITS        000000000000000 02371 000000 00      0   0  1
  [30] .eh_frame         PROGBITS        00000000000d904 00d904 000474 00   A  0   0  8
```

---

## 6. 节区类型

### 6.1 .text 节区

**类型**: SHT_PROGBITS
**标志**: SHF_ALLOC | SHF_EXECINSTR

代码节区，存储可执行指令。

### 6.2 .data 和 .data.rel.ro 节区

**类型**: SHT_PROGBITS
**标志**: SHF_ALLOC | SHF_WRITE (或 SHF_ALLOC 用于 .data.rel.ro)

已初始化数据节区。

### 6.3 .bss 节区

**类型**: SHT_NOBITS
**标志**: SHF_ALLOC | SHF_WRITE

未初始化数据节区，不占用文件空间，运行时分配并初始化为零。

### 6.4 .rodata 节区

**类型**: SHT_PROGBITS
**标志**: SHF_ALLOC

只读数据节区。

### 6.5 .init 和 .fini 节区

**类型**: SHT_PROGBITS
**标志**: SHF_ALLOC | SHF_EXECINSTR

初始化和结束函数。

### 6.6 .plt 和 .got 节区

**类型**: SHT_PROGBITS
**标志**: SHF_ALLOC | SHF_EXECINSTR (plt) / SHF_WRITE (got)

程序链接表 (Procedure Linkage Table) 和全局偏移表 (Global Offset Table)，用于动态链接。

### 6.7 .dynamic 节区

**类型**: SHT_DYNAMIC
**标志**: SHF_ALLOC | SHF_WRITE

动态链接信息。

### 6.8 .dynsym 节区

**类型**: SHT_DYNSYM
**标志**: SHF_ALLOC

动态链接符号表 (精简版符号表)。

### 6.9 .dynstr 节区

**类型**: SHT_STRTAB
**标志**: SHF_ALLOC

动态链接字符串表。

### 6.10 .symtab 节区

**类型**: SHT_SYMTAB
**标志**: 无 SHF_ALLOC

完整符号表 (用于链接，不用于运行时)。

### 6.11 .strtab 节区

**类型**: SHT_STRTAB

字符串表。

### 6.12 .shstrtab 节区

**类型**: SHT_STRTAB

节区名称字符串表。

### 6.13 .rel/.rela 节区

**类型**: SHT_REL / SHT_RELA
**标志**: SHF_ALLOC

重定位信息。

### 6.14 .init_array 和 .fini_array

**类型**: SHT_INIT_ARRAY / SHT_FINI_ARRAY
**标志**: SHF_ALLOC | SHF_WRITE

构造函数和析构函数数组。

---

## 7. 符号表 (Symbol Table)

### 7.1 Elf64_Sym 结构

```c
typedef struct {
    Elf64_Word    st_name;     /* 符号名称 (字符串表索引) */
    unsigned char st_info;    /* 符号类型和绑定 */
    unsigned char st_other;   /* 符号可见性 */
    Elf64_Half    st_shndx;    /* 所在节区索引 */
    Elf64_Addr    st_value;   /* 符号值 */
    Elf64_Xword   st_size;    /* 符号大小 */
} Elf64_Sym;
```

### 7.2 符号类型 (st_info & 0x0F)

| 值 | 名称 | 描述 |
|----|------|------|
| STT_NOTYPE | 0 | 无类型 |
| STT_OBJECT | 1 | 数据对象 |
| STT_FUNC | 2 | 函数 |
| STT_SECTION | 3 | 节区 |
| STT_FILE | 4 | 文件名 |
| STT_COMMON | 5 | 通用块 |
| STT_TLS | 6 | 线程局部存储 |
| STT_GNU_IFUNC | 10 | 间接函数 |

### 7.2.1 GNU IFUNC (间接函数)

GNU IFUNC是一种运行时多态机制，允许函数实现根据运行平台动态选择：

**特性**:
- 符号类型为 `STT_GNU_IFUNC`
- 指向一个解析器函数，而非最终实现
- 动态链接器在进程启动时调用解析器
- 用于根据CPU特性选择最优实现（如memcpy, strlen等）

**示例使用**:
```c
// IFUNC解析器函数
static void *my_strchr_resolver(void) {
    // 检测CPU特性并返回最佳实现
    if (CPU_HAS_SSE42)
        return strchr_sse42;
    else
        return strchr_generic;
}

// 声明IFUNC符号
void *strchr(char *, int) __attribute__((ifunc("my_strchr_resolver")));
```

**编译方式**:
```asm
.type strchr, @gnu_indirect_function
```

**查看IFUNC**:
```bash
# 查看IFUNC符号
readelf -s /lib64/libc.so.6 | grep IFUNC

# nm显示
nm -C /lib64/libc.so.6 | grep IFUNC
```

### 7.3 符号绑定 (st_info >> 4)

| 值 | 名称 | 描述 |
|----|------|------|
| STB_LOCAL | 0 | 局部 |
| STB_GLOBAL | 1 | 全局 |
| STB_WEAK | 2 | 弱绑定 |
| STB_GNU_UNIQUE | 10 | 唯一全局 (GNU扩展) |

### 7.4 符号可见性 (st_other & 0x03)

| 值 | 名称 | 描述 |
|----|------|------|
| STV_DEFAULT | 0 | 默认 |
| STV_INTERNAL | 1 | 内部 |
| STV_HIDDEN | 2 | 隐藏 |
| STV_PROTECTED | 3 | 受保护 |

### 7.5 特殊节区索引

| 值 | 名称 | 描述 |
|----|------|------|
| SHN_UNDEF | 0 | 未定义符号 |
| SHN_ABS | 0xFFF1 | 绝对值符号 |
| SHN_COMMON | 0xFFF2 | 通用块符号 |
| SHN_LORESERVE | 0xFF00 | 保留索引起始 |
| SHN_XINDEX | 0xFFFF | 索引在扩展节区中 |

### 7.6 查看符号表

```bash
$ readelf -s /bin/ls | head -30
Symbol table '.dynsym' contains 706 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND putenv@GLIBC_2.2.5 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __uflow@GLIBC_2.2.5 (2)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __overflow@GLIBC_2.2.5 (2)
     ...
```

---

## 8. 字符串表 (String Table)

### 8.1 字符串表结构

字符串表是一个连续的字节序列，以 null 结尾的字符串数组。
- 第一个字节总是 null (空字符串)
- 索引0表示空字符串或"未命名"

### 8.2 字符串表布局示例

```
索引:  00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19
内容:  \0 h \0 e \0 l \0 l \0 o \0 w \0 o \0 r \0 l \0 d \0
字符串: "" "h" "e" "l" "l" "o" "w" "o" "r" "l" "d"
```

### 8.3 字符串表使用

- 节区名称: `.shstrtab` (节区头 sh_name 索引)
- 符号名称: `.dynstr` 或 `.strtab` (符号 st_name 索引)
- 文件名称: 编译时传入

---

## 9. 重定位表 (Relocation Table)

### 9.1 重定位概念

重定位是将符号引用与符号定义进行连接的过程。

### 9.2 Elf64_Rela 结构

```c
typedef struct {
    Elf64_Addr    r_offset;  /* 应用重定位的位置 */
    Elf64_Xword   r_info;     /* 符号索引和重定位类型 */
    Elf64_Sxword  r_addend;   /* 加数 */
} Elf64_Rela;
```

### 9.3 r_info 编码

```c
#define ELF64_R_SYM(i)    ((i) >> 32)
#define ELF64_R_TYPE(i)   ((i) & 0xffffffff)
#define ELF64_R_INFO(s,t) (((s) << 32) + (t))
```

### 9.4 x86-64 重定位类型

| 类型 | 名称 | 描述 |
|------|------|------|
| R_X86_64_NONE | 0 | 无操作 |
| R_X86_64_64 | 10 | 直接64位 |
| R_X86_64_PC32 | 2 | PC相对32位 |
| R_X86_64_GOT32 | 4 | GOT入口32位 |
| R_X86_64_PLT32 | 4 | PLT入口相对32位 |
| R_X86_64_GOTPCREL | 9 | GOT相对PC |
| R_X86_64_COPY | 5 | 复制符号 |
| R_X86_64_GLOB_DAT | 6 | 全局数据符号 |
| R_X86_64_JUMP_SLOT | 7 | 过程链接表条目 |
| R_X86_64_RELATIVE | 8 | 相对地址 |
| R_X86_64_IRELAX | 51 | 宽松IRELAX |

### 9.5 RELR (Relative Relocations)

RELR是一种高效的相对重定位编码格式，用于减少动态库中的重定位条目数量。

#### 9.5.1 RELR编码原理

RELR使用位图编码相对重定位地址：
- 第一个条目是显式地址（基准地址）
- 后续位图位表示哪些相邻地址有相对重定位

```c
// RELR entry: 64-bit address
typedef Elf64_Addr Elf64_Relr;

// Encoding: odd addresses indicate bitmap, even are single addresses
// Bitmap: each set bit indicates (address + bit_index * sizeof(Elf64_Addr))
```

#### 9.5.2 RELR动态标签

| 标签 | 描述 |
|------|------|
| DT_RELR | RELR重定位表地址 |
| DT_RELRSZ | RELR表大小（字节） |
| DT_RELRENT | RELR条目大小 |

#### 9.5.3 RELR优势

- **压缩效率**: 相比RELA，可节省50-70%空间
- **仅相对地址**: 只能表示 `S + A - P` 类型的重定位
- **编码限制**: 无法表示奇数地址（需要额外的处理）

#### 9.5.4 查看RELR

```bash
# 查看RELR重定位
readelf --relocs /lib64/libc.so.6 | grep RELR

# 查看动态段中的RELR标签
readelf -d /lib64/libc.so.6 | grep -i relr
```

### 9.6 CREL (Compact Relocation Format)

CREL是正在开发中的新型紧凑重定位格式，旨在进一步减少重定位开销。

#### 9.6.1 CREL设计目标

- 比RELR更高的压缩率
- 支持更广泛的重定位类型
- 保持解析效率

#### 9.6.2 CREL结构

```c
// SHT_CREL (新节区类型)
// Elf64_Crel entry
typedef struct {
    Elf64_Addr  cr_offset;    // 重定位偏移
    Elf64_Xword cr_info;      // 类型和符号索引
} Elf64_Crel;
```

#### 9.6.3 CREL状态

- **LLVM**: 正在实现中
- **GCC**: 讨论中
- **规范**: 尚未最终确定

### 9.7 查看重定位表

```bash
$ readelf -r /bin/ls | head -30
Relocation section '.rela.dyn' at offset 0x6880 contains 405 entries:
  Offset          Type           Sym. Name    + Addend
0000000000000000 R_X86_64_RELATIVE              + 0
0000000000000008 R_X86_64_GLOB_DAT  __gmon_start__ + 0
0000000000000010 R_X86_64_GLOB_DAT  _ITM_deregisterT. + 0
0000000000000018 R_X86_64_GLOB_DAT  _ITM_registerT. + 0
0000000000000020 R_X86_64_GLOB_DAT  __cxa_finalize@GLIBC_2.2.5 + 0
...
```

---

## 10. 动态链接

### 10.1 动态链接概述

动态链接允许可执行文件在运行时加载共享库，实现代码共享和模块化。

### 10.2 动态段结构

```c
typedef struct {
    Elf64_Sxword  d_tag;    /* 动态条目类型 */
    union {
        Elf64_Xword d_val;  /* 数值 */
        Elf64_Addr  d_ptr;  /* 指针 */
    } d_un;
} Elf64_Dyn;
```

### 10.3 动态标签类型 (d_tag)

| 类型 | 描述 |
|------|------|
| DT_NULL | 列表结束 |
| DT_NEEDED | 需要的共享库名称 (d_val) |
| DT_SONAME | 共享库名称 (d_val) |
| DT_RPATH | 运行时库搜索路径 (d_val) |
| DT_RUNPATH | 运行时库搜索路径 (d_val) |
| DT_SYMTAB | 动态符号表地址 (d_ptr) |
| DT_STRTAB | 动态字符串表地址 (d_ptr) |
| DT_STRSZ | 字符串表大小 (d_val) |
| DT_SYMENT | 符号表项大小 (d_val) |
| DT_PLTRELSZ | PLT重定位表大小 (d_val) |
| DT_PLTGOT | PLT/GOT地址 (d_ptr) |
| DT_HASH | 符号哈希表地址 (d_ptr) |
| DT_GNU_HASH | GNU风格哈希表地址 (d_ptr) |
| DT_REL / DT_RELA | 动态重定位表地址 (d_ptr) |
| DT_RELSZ / DT_RELASZ | 重定位表大小 (d_val) |
| DT_JMPREL | PLT重定位表地址 (d_ptr) |
| DT_INIT | 初始化函数地址 (d_ptr) |
| DT_FINI | 结束函数地址 (d_ptr) |
| DT_PREINIT_ARRAY | 预初始化数组地址 (d_ptr) |
| DT_INIT_ARRAY | 初始化数组地址 (d_ptr) |
| DT_FINI_ARRAY | 结束数组地址 (d_ptr) |

### 10.4 动态链接过程

1. **加载**: 内核加载ELF，将PT_LOAD段映射到内存
2. **解析依赖**: 读取DT_NEEDED，加载依赖的共享库
3. **符号解析**: 使用DT_HASH/DT_GNU_HASH查找符号
4. **重定位**:
   - DT_REL/DT_RELA: 应用基址重定位
   - DT_JMPREL: PLT延迟绑定
5. **初始化**: 调用DT_INIT_ARRAY中的函数

### 10.5 PLT (Procedure Linkage Table)

PLT是函数调用的跳转表，首次调用时触发符号解析：

```asm
# PLT条目伪代码
plt_entry:
    jmp [GOT_entry]      # 跳转到GOT存储的地址
    push reloc_index    # 压入重定位索引
    jmp PLT0            # 跳转到链接器桩代码
```

### 10.6 GOT (Global Offset Table)

GOT存储全局变量和函数地址：
- .got.plt: PLT使用的GOT
- .got: 位置无关数据

### 10.7 lazy binding

默认情况下，符号解析延迟到第一次调用时进行，通过GOT和PLT实现。

### 10.8 查看动态段

```bash
$ readelf -d /bin/ls | head -30

Dynamic section at offset 0x15eb48 contains 28 entries:
  Tag        Type                 Name/Value
 0x0000000000000001 (NEEDED)        Shared library: [libcap.so.2]
 0x0000000000000001 (NEEDED)        Shared library: [libc.so.6]
 0x000000000000000c (INIT)          0x1900
 0x000000000000000d (FINI)          0xd4d0
 0x0000000000000019 (RELA)          0x6880
 0x000000000000001a (RELASZ)        149688 (bytes)
 0x000000000000001b (RELAENT)       24 (bytes)
 0x000000000000001e (FLAGS)         BIND_NOW
 0x000000000000001e (FLAGS)         ORIGIN
 0x000000000000001f (SYMBOLIC)      0x0
 ...
```

---

## 10.5 核心转储文件 (Core Dump)

### 10.5.1 核心转储概述

核心转储文件 (ET_CORE) 是进程崩溃时生成的内存快照，格式也是ELF：

```bash
# 生成核心转储
ulimit -c unlimited
./program  # 崩溃后生成 core 文件

# 查看核心转储
gdb ./program core
```

### 10.5.2 核心转储结构

| 节区 | 说明 |
|------|------|
| ET_CORE | 文件类型为ET_CORE |
| PT_NOTE | 进程信息 (寄存器状态、线程信息) |
| PT_LOAD | 进程内存映射 |
| .note | 构建ID、ABI信息 |

### 10.5.3 核心转储程序头

```bash
$ readelf -l core
Elf file type is CORE (file type value 4)

Program Headers:
  Type           Offset   VirtAddr           FileSiz  MemSiz   Flg Align
  NOTE           0x000268 0x0000000000000000 0x000558 0x000558 R   0x1
  LOAD           0x001000 0x0000000000400000 0x1a000 0x1a000 R E 0x1000
  LOAD           0x1b000 0x0000000000600000 0x01000 0x03000 RW  0x1000
```

### 10.5.4 .note节区结构

.note节区用于存储元数据，如ABI信息、构建ID等：

| 字段 | 大小 | 描述 |
|------|------|------|
| n_namesz | 4 | 名称长度 |
| n_descsz | 4 | 描述长度 |
| n_type | 4 | 类型 |
| name | 名称 | 填充到4字节对齐 |
| desc | 描述 | 实际数据 |

### 10.5.5 常见.note类型

| 类型 | 说明 |
|------|------|
| NT_PRSTATUS | 通用寄存器状态 |
| NT_PRFPREG | 浮点寄存器状态 |
| NT_PRPSINFO | 进程信息 |
| NT_TASKSTRUCT | 任务结构 |
| NT_AUXV | 辅助向量 |
| NT_GNU_BUILD_ID | 构建ID (唯一标识) |
| NT_GNU_GOLD_VERSION | gold链接器版本 |

### 10.5.6 .note.ABI-tag

.note.ABI-tag声明预期的运行时ABI：

```bash
$ readelf -n /bin/ls
Displaying notes found in the .note.ABI-tag section:
  Owner         Data size    Description
  GNU          0x00000010    NT_GNU_ABI_TAG (ABI version tag)
    OS: Linux, ABI: 3.2.0
```

### 10.5.7 核心转储配置

```bash
# 查看当前配置
cat /proc/sys/kernel/core_pattern

# 设置核心转储文件名模板
echo "/tmp/core.%e.%p" > /proc/sys/kernel/core_pattern

# 使用systemd-coredump
systemctl enable systemd-coredump

# 禁用核心转储
echo "0" > /proc/sys/kernel/core_pattern
ulimit -c 0
```

### 10.5.8 查看核心转储

```bash
# 使用readelf查看note信息
readelf -n core

# 使用gdb分析
gdb ./program core

# 使用coredumpctl (systemd)
coredumpctl dump <pid>

# 使用objdump
objdump -s -j .note core
```

---

## 10.6 动态链接器 (rtld)

### 10.6.1 动态链接器概述

动态链接器 (rtld, /lib64/ld-linux-x86-64.so.2) 是Linux内核加载的第一个用户空间程序，负责：

- 加载所有需要的共享库
- 解析符号
- 应用重定位
- 初始化程序

### 10.6.2 PT_INTERP段

PT_INTERP指定动态链接器路径：

```bash
$ readelf -l /bin/ls | grep INTERP
  INTERP         0x000238 0x0000000000000238 0x0000000000000238 0x00001c 0x00001c R   0x1
    [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

### 10.6.3 动态链接器环境变量

| 变量 | 说明 |
|------|------|
| LD_LIBRARY_PATH | 库搜索路径 |
| LD_PRELOAD | 预加载共享库 |
| LD_DEBUG | 调试信息 (libs, bindings, sections) |
| LD_BIND_NOW | 立即绑定 (禁用lazy binding) |
| LD_BIND_NOT | 不更新GOT |
| LD_ORIGIN | $ORIGIN路径 |

**示例**:
```bash
# 查看动态链接详情
LD_DEBUG=libs /bin/ls

# 查看符号绑定
LD_DEBUG=bindings /bin/ls 2>&1 | grep puts
```

### 10.6.4 Linux内核模块

Linux内核模块(.ko文件)也是ELF格式，用于动态扩展内核功能。

#### 10.6.4.1 内核模块结构

```bash
# 查看模块信息
modinfo /lib/modules/$(uname -r)/kernel/drivers/net/tun.ko

# 查看模块依赖
lsmod

# 反汇编内核模块
objdump -d /lib/modules/$(uname -r)/kernel/drivers/net/tun.ko
```

#### 10.6.4.2 内核模块节区

| 节区 | 描述 |
|------|------|
| .text | 模块代码 |
| .data | 模块数据 |
| .rodata | 只读数据 (如版本字符串) |
| .modinfo | 模块元数据 |
| .gnu.linkonce.this_module | 模块结构 |

#### 10.6.4.3 内核模块 vs 用户空间ELF

| 特性 | 内核模块 | 用户空间ELF |
|------|----------|-------------|
| 运行特权级 | Ring 0 (内核) | Ring 3 (用户) |
| libc依赖 | 无 | glibc等 |
| 入口点 | init_module() | main() |
| 出口点 | cleanup_module() | atexit() |
| 加载器 | kernel/module.c | ld-linux.so |
| 可执行标志 | 通常无 | 有 |

### 10.6.5 链接器搜索路径

动态链接器按以下顺序搜索库：

1. DT_RPATH (已废弃，gcc 10+默认禁用)
2. LD_LIBRARY_PATH (环境变量)
3. DT_RUNPATH (推荐使用)
4. /etc/ld.so.cache
5. /lib, /lib64, /usr/lib 等默认路径

**DT_RPATH vs DT_RUNPATH**:
- `-Wl,-rpath,PATH` 设置DT_RUNPATH（推荐）
- `-Wl,--disable-new-dtags` 可启用旧的DT_RPATH行为
- RUNPATH允许运行时覆盖，不影响其他程序

### 10.6.5 LD_PRELOAD机制

LD_PRELOAD允许在所有其他库之前加载指定共享库：

```bash
# 预加载库
LD_PRELOAD=/lib64/libpthread.so.0 ./program

# 查看预加载的库
ldd /program

# 禁用LD_PRELOAD
LD_PRELOAD= ./program
```

**常见用途**:
- 性能分析工具 (e.g., lib annotating profiling)
- 内存检测工具 (e.g., valgrind)
- 劫持系统函数进行调试

### 10.6.6 符号绑定与覆盖

动态链接器符号解析顺序（当多个库定义同一符号时）：

1. **RTLD_GLOBAL**标志的库中符号优先
2. 可执行文件中的符号优先
3. 按加载顺序搜索DT_NEEDED依赖树
4. LD_PRELOAD的库（最先搜索）

**符号冲突处理**:
```bash
# 查看符号定义位置
nm -C /lib64/libc.so.6 | grep printf

# 使用objdump追踪符号
objdump -T /lib64/libc.so.6 | grep -E "(puts|printf)"
```

### 10.6.7 RTLD (Run-Time Linker) 内部机制

动态链接器工作流程详解：

```c
// 简化流程
1. parse_elf()        // 解析ELF头和程序头
2. map_segments()     // 映射PT_LOAD段到内存
3. reloc_interp()     // 定位并调用解释器
4. load_deps()        // 递归加载依赖库
5. symbol_resolve()   // 解析所有符号引用
6. reloc_apply()      // 应用重定位
7. call_init()        // 调用.init_array函数
```

**延迟绑定 (Lazy Binding)**:
- PLT条目首次调用时触发符号解析
- GOT条目被动态链接器填充
- 后续调用直接跳转到已解析地址

**立即绑定 (Eager Binding)**:
- `LD_BIND_NOW=1` 禁用延迟绑定
- 程序启动时完成所有符号解析
- 更长的启动时间，但运行时更快更可预测

---

## 11. 线程局部存储 (TLS)

### 11.1 TLS概述

线程局部存储 (Thread Local Storage, TLS) 是一种机制，允许每个线程拥有变量的独立副本。ELF格式通过专门的节区和程序头部来支持TLS。

### 11.2 TLS相关节区

| 节区 | 类型 | 描述 |
|------|------|------|
| .tdata | SHT_PROGBITS | TLS初始化数据 (占用文件空间) |
| .tbss | SHT_NOBITS | TLS未初始化数据 (不占用文件空间) |
| .thread_common | SHT_TLS | 通用TLS块 |

### 11.3 TLS节区标志

TLS节区使用 `sh_flags` 中的 `SHF_TLS` (0x400) 标志标识。

### 11.4 TLS模板布局

```
+------------------+
| .tdata           |  TLS初始化数据
| (SHF_TLS+ALLOC)  |
+------------------+
| .tbss            |  TLS未初始化数据
| (SHF_TLS+NOBITS) |
+------------------+
```

### 11.5 TLS动态链接

TLS相关的动态链接条目：

| 动态标签 | 描述 |
|----------|------|
| DT_TLSDESC_GOT | TLS描述符GOT条目 |
| DT_TLSDESC_PLT | TLS描述符PLT条目 |
| DT_TLS_MODULE_ID | TLS模块ID |
| DT_TLS_OFFSET | TLS偏移量 |

### 11.6 TLS访问模型

x86-64支持多种TLS访问模型：

| 模型 | 说明 | 指令序列 |
|------|------|----------|
| GD (Global Dynamic) | 动态TLS，符号尚未知 | `leaq var@TLSGD(%rip), %rdi; call __tls_get_addr@PLT` |
| LD (Local Dynamic) | 本地TLS，可优化 | `leaq var@TLSLD(%rip), %rdi; call __tls_get_addr@PLT` |
| IE (Initial Exec) | 静态已知TLS | `movq %fs:0x0, %rax; addq var@GOTTPOFF(%rip), %rax` |
| LE (Local Exec) | 静态TLS | `movq %fs:0x0, %rax; addq var@TPOFF(%rax), %rax` |

### 11.7 查看TLS

```bash
# 查看TLS节区
readelf -S /lib64/libc.so.6 | grep -i tls

# 查看TLS段
readelf -l /lib64/libc.so.6 | grep -i tls

# objdump查看TLS
objdump -d -M intel /lib64/libc.so.6 | grep -A5 tls
```

---

## 12. 安全机制

### 12.1 安全概述

现代ELF文件包含多种安全机制来防御 exploits 和攻击。

### 12.2 RELRO (Relocation Read-Only)

RELRO是一种将重定位区域标记为只读的安全机制：

| 类型 | 描述 | 标志 |
|------|------|------|
| Partial RELRO | 部分重定位只读 | 默认开启 |
| Full RELRO | 全部重定位只读 | -Wl,-z,relro,-z,now |

**Full RELRO效果：**
- 所有重定位在程序启动时完成
- .got.plt变为只读
- 防止GOT覆写攻击

### 12.3 ASLR (Address Space Layout Randomization)

ASLR通过随机化内存布局来阻止攻击：

| 组件 | 随机化 | 可控 |
|------|--------|------|
| 基址 | 随机 | 0x55xxxxxx |
| 栈 | 随机 | 16字节对齐 |
| 堆 | 随机 | glibc 2.18+ |
| VDSO | 随机 | 页面对齐 |

**检查ASLR：**
```bash
# 查看程序头部中的对齐要求
readelf -l /bin/ls | grep LOAD
# 对齐0x200000表示支持随机化

# 查看ASLR状态
cat /proc/sys/kernel/randomize_va_space
```

### 12.4 DEP/NX (Data Execution Prevention)

DEP (也称 NX/No-Execute) 防止从数据页执行代码：

| 标志 | 描述 |
|------|------|
| PF_X (可执行) | 代码页设置 |
| PF_W (可写) | 数据页设置 |
| 组合 | 不可同时可写可执行 |

### 12.5 Stack Canaries (栈保护)

栈金丝雀检测栈溢出：

| 类型 | 符号 | 说明 |
|------|------|------|
| 经典 | __stack_chk_fail | GCC -fstack-protector |
| 强 | __stack_chk_fail_local | GCC -fstack-protector-strong |
| 所有 | __stack_chk_fail | GCC -fstack-protector-all |

### 12.6 FORTIFY_SOURCE

编译时和运行时检查：

| 检查 | 宏 | 说明 |
|------|-----|------|
| 边界 | _FORTIFY_SOURCE=2 | 编译时缓冲检查 |
| 运行时 | __fortify_fail | 运行时检测 |

### 12.7 PIE (Position Independent Executable)

PIE允许可执行文件像共享库一样被加载：

```bash
# 编译PIE可执行文件
gcc -fPIE -pie -o hello hello.c

# 检查是否为PIE
readelf -h hello | grep Type
# Type: DYN (Shared object file) - PIE
# Type: EXEC (Executable file) - 非PIE
```

### 12.8 CET (Control Flow Enforcement Technology)

Intel CET提供硬件级别的控制流保护：

| 特性 | 描述 | ELF标志 |
|------|------|---------|
| IBT (Indirect Branch Tracking) | 间接分支跟踪 | GNU_PROPERTY_X86_FEATURE_1_IBT |
| SHSTK (Shadow Stack) | 影子栈 | GNU_PROPERTY_X86_FEATURE_1_SHSTK |

**Shadow Stack (SHSTK)**:
- 防御ROP (Return-Oriented Programming) 攻击
- 影子栈是独立于普通栈的只读内存区域
- 保存返回地址的副本
- 内核Linux 5.9+支持，用户空间需要HW支持

**IBT (Indirect Branch Tracking)**:
- 防御CFI (Control Flow Integrity) 攻击
- 要求间接分支目标必须是ENDBR指令
- gcc -fcf-protection=full 启用

**兼容性要求**:
- Linux内核 5.9+ (早期内核通过向后兼容支持)
- Binutils 2.29+ 或 LLVM 6+
- HW支持 (Intel CET 或 AMD Shadow Stack)
- 运行时可通过 `no_ibt` 或 `no_shstk` 禁用

### 12.9 GNU_PROPERTY段

程序头部中的GNU_PROPERTY段声明安全特性：

```c
typedef struct {
    Elf64_Word   p_type;   // PT_GNU_PROPERTY (0x6474e553)
    // ...
} Elf64_Prop;
```

### 12.10 安全检查命令

```bash
# 检查RELRO
readelf -d binary | grep RELRO

# 检查PIE
readelf -h binary | grep Type

# 检查NX
readelf -l binary | grep GNU_STACK

# 检查所有安全特性
scanelf -l binary
# 或
readelf -a binary | grep -E "(STACK|COMMENT|GNU_PROPERTY)"

# 使用pahint检查
file binary
```

---

## 13. DWARF调试信息

### 13.1 DWARF概述

DWARF是一种调试信息格式，与ELF紧密集成，用于调试器定位源代码与机器码的对应关系。

### 13.2 DWARF相关节区

| 节区 | 描述 |
|------|------|
| .debug_info | 编译单元 DIE (Debugging Information Entry) |
| .debug_abbrev | DIE属性 abbreviation 表 |
| .debug_line | 行号信息 |
| .debug_frame | 调用帧信息 ( unwind ) |
| .debug_str | 调试字符串 |
| .debug_loc | 位置描述 |
| .debug_ranges | 地址范围 |
| .debug_types | 类型信息 (DWARF4前) |
| .debug_macinfo | 宏信息 |

### 13.3 DWARF版本

| 版本 | 年份 | 主要特性 |
|------|------|----------|
| DWARF 2 | 1993 | 基础调试信息 |
| DWARF 3 | 2006 | 模板、命名空间 |
| DWARF 4 | 2010 | 单元分割、压缩 |
| DWARF 5 | 2017 | 改进的索引、_split_debug_file、字符串表索引 |
| DWARF 6 | 2021 | UTF-8支持改进、debug_line表压缩 |

**DWARF 5主要改进**:
- 改进的Debug Information Entry (DIE)压缩
- 独立的字符串表节区(.debug_str_offsets)
- 改进的Macro信息节区(.debug_macinfo)
- 新增.debug_line_str节区(行号字符串表)
- 新增.debug_addr节区(地址表)
- 支持split DWARF (.dwo/.dwp文件)

**DWARF 6新特性** (2021年发布):
- 改进的UTF-8字符串支持
- 新的调试信息属性

### 13.4 行号表结构

```c
// DWARF行号表条目
typedef struct {
    Dwarf_Addr addr;     // 指令地址
    Dwarf_Unsigned line; // 源文件行号
    Dwarf_Unsigned col;   // 列号
} Dwarf_Line;
```

### 13.5 查看DWARF信息

```bash
# 查看行号信息
readelf --debug-dump=line /bin/ls

# 查看符号对应的行号
addr2line -e /bin/ls 0x401000

# 查看函数信息
readelf --debug-dump=info /bin/ls | head -100

# objdump显示行号
objdump -d -l /bin/ls

# 使用dwarfdump
dwarfdump -i /bin/ls

# 查看DWARF版本
readelf --debug-dump=info /bin/ls | head -5

# 查看DWARF 5的split debug
readelf --debug-dump=info --executable /bin/ls

# 使用llvm-dwarfdump (LLVM工具链)
llvm-dwarfdump -a /bin/ls

# 查看所有DWARF节区
readelf -S /bin/ls | grep debug
```

### 13.6 .eh_frame节区

.eh_frame用于异常处理和栈展开，基于DWARF调用帧信息 (CFI)：

| 表条目 | 描述 |
|--------|------|
| CIE (Common Information Entry) | 公共信息，每个被调用函数一个 |
| FDE (Frame Description Entry) | 帧描述，每个函数实例一个 |

### 13.6.1 CIE和FDE结构

**CIE (Common Information Entry)**:
- 包含帧状态初始状态
- 保存规则 (如何保存寄存器)
- 初始指令

**FDE (Frame Description Entry)**:
- 指向关联的CIE
- PC范围描述
- 恢复规则 (如何恢复寄存器)

### 13.6.2 .eh_frame vs .debug_frame

| 特性 | .eh_frame | .debug_frame |
|------|-----------|--------------|
| 用途 | 运行时异常处理 | 调试器栈展开 |
| 位置 |  alloc'd 节区 | non-alloc'd 节区 |
| 解析器 | libgcc | 调试器 |
| 内容 | 每个函数一个FDE | 类似但无初始化 |

### 13.6.3 栈展开过程

1. 异常抛出时，_Unwind_* 函数被调用
2. 遍历.eh_frame中的FDE
3. 对每个帧执行DWARF字节码
4. 恢复寄存器状态
5. 找到最近的catch处理器

### 13.6.4 查看.eh_frame

```bash
# 查看.eh_frame内容
readelf -w /bin/ls

# objdump显示
objdump -s -j .eh_frame /bin/ls

# llvm-readelf
llvm-readelf -w /bin/ls

# 使用unwinder验证
# libunwind项目
```

### 13.6.5 .eh_frame_hdr

.eh_frame_hdr是.eh_frame的索引，加速查找：

```bash
$ readelf -h /bin/ls | grep -A2 eh_frame
```

---

### 13.7 调试器交互

```bash
# gdb查看函数行号
gdb -batch -ex "info line *0x401000" /bin/ls

# 查看当前函数
gdb -batch -ex "info symbol 0x401000" /bin/ls
```

### 13.8 Split DWARF (Debug Fission)

Split DWARF允许将调试信息分离到单独的 .dwo 文件中：

| 文件 | 说明 |
|------|------|
| .o | 目标文件 (含部分.debug_*节区) |
| .dwo | 编译单元的完整调试信息 |
| .dwp | 打包的.dwo文件 (多个.dwo合并) |

### 13.8.1 Split DWARF优势

- **减小最终二进制大小**: 调试信息不在最终可执行文件中
- **加速链接**: 链接器不需要处理大量调试信息
- **并行编译**: 调试信息可以并行生成

### 13.8.2 编译选项

```bash
# GCC: 生成split DWARF
gcc -gsplit-dwarf -c source.c

# Clang: 同样支持
clang -gsplit-dwarf source.c

# 查看生成的.dwo文件
ls -la source.dwo
```

### 13.8.3 打包DWARF包

```bash
# 创建.dwp包
dwp -o libfoo.dwp libfoo1.dwo libfoo2.dwo

# 查看.dwp内容
llvm-dwarfdump libfoo.dwp
```

### 13.8.4 使用Debug Link

二进制文件通过 .gnu_debuglink 节区引用调试文件：

```bash
# 查看调试链接
readelf -Wn /bin/ls

# objdump显示
objdump -s -j .gnu_debuglink /bin/ls
```

### 13.8.5 GNU Build ID

Build ID用于自动查找对应的调试文件：

```bash
# 查看Build ID
readelf -n /bin/ls | grep Build

# 调试文件路径约定
/usr/lib/debug/.build-id/xx/yyyy.debug
#                    └─ Build ID前2字符
#                       └─ Build ID其余字符
```

### 13.8.6 调试文件查找

GDB自动查找调试文件：

```bash
# 设置调试文件目录
set debug-file-directory /usr/lib/debug

# 手动加载调试文件
gdb -ex "symbol-file /path/to/debug/file"

# 使用.debug文件的路径
add-symbol-file /path/to/executable.debug
```

---

## 14. 压缩节区

### 14.1 压缩节区概述

ELF支持压缩节区以减小文件大小和加载时间。

### 14.2 SHF_COMPRESSED标志

```c
#define SHF_COMPRESSED  0x800000    // 节区被压缩
```

### 14.3 压缩节区格式

压缩节区在 `sh_flags` 中包含 `SHF_COMPRESSED`，内容格式：

| 字段 | 大小 | 描述 |
|------|------|------|
| ch_type | 4/8字节 | 压缩类型 (ELFCOMPRESS_ZLIB等) |
| ch_size | 4/8字节 | 压缩前大小 |
| ch_addralign | 4/8字节 | 对齐要求 |

### 14.4 压缩类型

| 类型 | 值 | 说明 |
|------|-----|------|
| ELFCOMPRESS_ZLIB | 1 | zlib压缩 (最常用) |
| ELFCOMPRESS_ZSTD | 2 | zstd压缩 (现代高效) |
| ELFCOMPRESS_LOOS | 0xFF000000 | OS特定起始 |
| ELFCOMPRESS_HIOS | 0xFFFF0000 | OS特定结束 |
| ELFCOMPRESS_LOPROC | 0xFF000000 | 处理器特定起始 |
| ELFCOMPRESS_HIPROC | 0xFFFF0000 | 处理器特定结束 |

**zstd压缩支持** (LLVM 15+/binutils 2.39+):
- 更高的压缩比和更快的解压速度
- 通过 `--compress-debug-sections=zstd` 启用
- lld支持并行压缩 (`--compress-sections`)

### 14.5 常见压缩节区

| 节区 | 压缩 | 说明 |
|------|------|------|
| .debug_str | ZLIB/ZSTD | 调试字符串 |
| .debug_line | ZLIB/ZSTD | 行号表 |
| .debug_info | ZLIB/ZSTD | DIE信息 |
| .debug_abbrev | ZLIB/ZSTD | 缩写表 |
| .comment | ZLIB | 注释 |
| .rodata.str* | ZLIB | 只读字符串 |

### 14.6 查看压缩节区

```bash
# 查看压缩信息
readelf -S /bin/ls | grep -i compress

# 查看详细压缩数据
readelf --hex-dump=.debug_str /bin/ls

# file命令检测
file /bin/ls

# 查看压缩比
size -A /bin/ls
```

### 14.7 压缩工具

```bash
# 使用eu-strip移除压缩(调试)
eu-strip --remove-comment /bin/ls

# 使用objcopy处理压缩
objcopy --compress-debug-sections=zlib /bin/ls
objcopy --compress-debug-sections=zstd /bin/ls

# lld/LD链接时压缩
ld.lld --compress-sections=zstd -o output input.o

# 并行压缩(zlib)
ld.lld --compress-sections=zlib --zlib-compress-multi-obj

# 查看压缩统计
size -A -d /bin/ls
```

---

## 15. 符号版本控制

### 15.1 符号版本概述

符号版本允许同一符号有多个版本，实现ABI兼容和向后兼容。

### 15.2 版本定义节区

| 节区 | 类型 | 描述 |
|------|------|------|
| .gnu.version | SHT_GNU_versym | 符号版本表 |
| .gnu.version_r | SHT_GNU_verneed | 版本需求 |
| .gnu.version_d | SHT_GNU_verdef | 版本定义 |

### 15.3 版本符号表结构

```c
typedef struct {
    Elf64_Half    vda_name;   // 版本定义名称索引
    Elf64_Half    vda_next;   // 下一条目偏移
} Elf64_Verdef;

typedef struct {
    Elf64_Half    vna_hash;   // 名称哈希
    Elf64_Half    vna_flags;  // 标志
    Elf64_Half    vna_other;  // 其他
    Elf64_Half    vna_name;   // 依赖名称
    Elf64_Half    vna_next;   // 下一条目偏移
} Elf64_Verneed;
```

### 15.4 版本符号结构

```c
typedef struct {
    Elf64_Half    st_name;    // 符号名
    unsigned char st_info;   // 类型和绑定
    unsigned char st_other;   // 可见性
    Elf64_Half    st_shndx;   // 节区索引
    Elf64_Half    st_value;   // 值
    Elf64_Xword   st_size;    // 大小
} Elf64_Sym;
// 高位半字存储版本索引
```

### 15.5 版本绑定标志

| 标志 | 描述 |
|------|------|
| VER_NDX_LOCAL | 本地版本 |
| VER_NDX_GLOBAL | 全局版本 |
| VER_NDX_MAX | 最大版本索引 |

### 15.6 查看符号版本

```bash
# 查看版本符号
readelf --dyn-syms /lib64/libc.so.6 | head -20

# 查看版本需求
readelf -V /lib64/libc.so.6

# 查看特定符号版本
objdump -T /lib64/libc.so.6 | grep -E "GLIBC_2\."
```

### 15.7 GNU_HASH

GNU_HASH提供更快的符号查找：

| 特性 | DT_HASH | DT_GNU_HASH |
|------|---------|-------------|
| 查找算法 | 链式哈希 | Bloom filter + 链式 |
| 符号顺序 | 按哈希排序 | 按哈希排序 |
| 桶数量 | 可变 | 2的幂 |
| 速度 | O(n) | O(1)平均 |

```bash
# 查看哈希表
readelf --dyn-syms --use-dynamic /lib64/libc.so.6

# 比较哈希实现
readelf -S /lib64/libc.so.6 | grep hash
```

---

## 15.7 链接器松弛 (Linker Relaxation)

### 15.7.1 链接器松弛概述

链接器松弛是一种链接时优化技术，通过用更短的指令序列替换代码来减小最终二进制大小或提高性能。

### 15.7.2 RISC-V链接器松弛

RISC-V是最积极使用链接器松弛的架构：

**常见松弛类型**:

| 原始序列 | 松弛后 | 节省 |
|----------|--------|------|
| auipc + lw | lw (pc-rel) | 4字节 |
| auipc + addi | addiw (pc-rel) | 4字节 |
| jal + jalr | jal (短跳转) | 2字节 |

**必需的重定位**:
```asm
# R_RISCV_RELAX 标记可松弛的指令
# R_RISCV_CALL_PLT 调用PLT条目
```

**查看松弛**:
```bash
# 查看重定位后的指令
objdump -dr a.out | less

# 使用 --emit-reloc
llvm-objdump -d --emit-relocs a.out
```

### 15.7.3 x86-64链接器松弛

**常见松弛类型**:

| 序列 | 松弛条件 |
|------|----------|
| movabsq → movq | 地址可编码为32位 |
| jmpq → jmp | 跳转目标在±2GB内 |
| RIP-relative → indirect | 符号局部性已知 |

**R_X86_64_PCREL_OPT**:
- 标记可优化的PC相对引用
- 允许链接器选择最优编码

### 15.7.4 其他架构

**ARM64 (AArch64)**:
- ADR → ADR (短偏移)
- 条件分支优化

**PowerPC64**:
- R_PPC64_PCREL_OPT
- TOC访问优化

### 15.7.5 链接器选项

```bash
# 启用松弛 (默认)
ld -z relax

# 禁用松弛
ld -z norelax

# 查看松弛信息
ld.lld --print-gc-sections a.out

# RISC-V特定
ld.lld --relax=true --emit-relocs
```

### 15.7.6 松弛的副作用

**潜在问题**:
- --emit-relocs保留原始重定位，可能与松弛冲突
- 调试信息可能不准确
- 地址无关代码(PIC)松弛更复杂

**最佳实践**:
```bash
# 发布构建 - 启用所有优化
ld.lld -O2 --relax -z relax

# 调试构建 - 禁用松弛
ld.lld -O0 -z norelax
```

---

## 16. 平台特定扩展

### 16.1 x86-64 (EM_X86_64)

**特定重定位类型：**

| 类型 | 描述 |
|------|------|
| R_X86_64_64 | 64位直接地址 |
| R_X86_64_PLT32 | PLT相对32位 |
| R_X86_64_GOTPCREL | GOT相对PC |
| R_X86_64_GOTPCREL_NX | GOT相对PC (Negate) |
| R_X86_64_TPOFF64 | TLS偏移64位 |
| R_X86_64_DTPOFF64 | TLS动态偏移64位 |

**x86-64特定标志：**

| 标志 | 值 | 描述 |
|------|-----|------|
| EF_X86_64_1 | 1 | 64位 |
| EF_X86_64_2 | 2 | 支持x32 ABI |
| EF_X86_64_3 | 4 | 支持富人区(FPO) |

### 16.2 x32 ABI (EM_X86_64 + 32位模式)

x32允许使用32位指针的64位代码：
```bash
gcc -mx32 -o hello hello.c
```

### 16.3 ARM (EM_ARM, 32位)

**ARM特定重定位：**

| 类型 | 描述 |
|------|------|
| R_ARM_ABS32 | 32位绝对地址 |
| R_ARM_PLT32 | PLT条目 |
| R_ARM_GOTOFF | GOT相对 |
| R_ARM_TLS_GD32 | TLS Global Dynamic |
| R_ARM_THM_CALL | Thumb调用 |

**ARM特定标志：**

| 标志 | 描述 |
|------|------|
| EF_ARM_HASENTRY | 有入口点 |
| EF_ARM_SYMSORT | 符号排序 |
| EF_ARM_DYNSYMSORT | 动态符号排序 |

### 16.4 AArch64 (EM_AARCH64, ARM64)

**AArch64特定重定位：**

| 类型 | 描述 |
|------|------|
| R_AARCH64_ABS64 | 64位绝对 |
| R_AARCH64_PREL64 | PC相对64位 |
| R_AARCH64_TLSLE | TLS本地偏移 |
| R_AARCH64_CALL26 | 函数调用26位 |
| R_AARCH64_JUMP26 | 跳转26位 |

### 16.5 RISC-V (EM_RISCV)

**RISC-V特定重定位：**

| 类型 | 描述 |
|------|------|
| R_RISCV_64 | 64位地址 |
| R_RISCV_PCREL_HI20 | PC相对高20位 |
| R_RISCV_PCREL_LO12_I | PC相对低12位 (立即数) |
| R_RISCV_RVC_BRANCH | 压缩分支 |
| R_RISCV_TPREL_HI20 | TLS相对高20位 |
| R_RISCV_TPREL_LO12_I | TLS相对低12位 |

**RISC-V特定ELF头标志：**

```c
#define EF_RISCV_RVC           0x00000001  // RVC (压缩指令)
#define EF_RISCV_FLOAT_ABI     0x00000006  // 浮点ABI掩码
#define EF_RISCV_FLOAT_ABI_SOFT 0x00000000 // 软浮点
#define EF_RISCV_FLOAT_ABI_SINGLE 0x00000002 // 单精度
#define EF_RISCV_FLOAT_ABI_DOUBLE 0x00000004 // 双精度
```

### 16.6 MIPS (EM_MIPS)

**MIPS特定扩展：**

| 标志 | 描述 |
|------|------|
| EF_MIPS_NOREORDER | 不可重排序 |
| EF_MIPS_PIC | 位置无关代码 |
| EF_MIPS_CPIC | 传统PIC |
| EF_MIPS_ABI2 | N32/64 ABI |
| EF_MIPS_FP64 | 64位浮点 |

### 16.7 LoongArch (EM_LOONGARCH)

龙芯架构支持的ELF特性：

| 类型 | 描述 |
|------|------|
| R_LARCH_64 | 64位地址 |
| R_LARCH_B26 | 26位分支 |
| R_LARCH_TLS_LE | TLS本地偏移 |

### 16.8 跨平台检查

```bash
# 检查特定架构特性
readelf -h /bin/ls | grep Machine

# 检查架构特定标志
readelf -h /bin/ls | grep Flags

# 列出支持的架构
readelf -V /bin/ls
```

### 16.9 eBPF (Extended Berkeley Packet Filter)

eBPF程序以特殊节区形式存储在Linux内核或对象文件中。

#### 16.9.1 eBPF节区

| 节区 | 描述 |
|------|------|
| .text | BPF程序代码 |
| .maps | BPF映射表 |
| .rodata | 只读数据 |
| .bss | 未初始化数据 |

#### 16.9.2 eBPF程序类型

| 类型 | 描述 |
|------|------|
| BPF_PROG_TYPE_SOCKET_FILTER | 套接字过滤 |
| BPF_PROG_TYPE_KPROBE | 内核探针 |
| BPF_PROG_TYPE_TRACEPOINT | 跟踪点 |
| BPF_PROG_TYPE_XDP | 快速数据包处理 |
| BPF_PROG_TYPE_PERF_EVENT | 性能事件 |

#### 16.9.3 查看eBPF程序

```bash
# 查看eBPF节区
readelf -S /sys/kernel/tracing/my_bpf_module | grep -i bpf

# 使用bpftool
bpftool prog list

# 反汇编eBPF程序
llvm-objdump -d /sys/kernel/debug/tracing/events/.../bpf_prog
```

#### 16.9.4 BPF CO-RE

BPF Core (Compile Once - Run Everywhere) 允许eBPF程序跨内核版本移植：

- 使用结构体字段访问而非直接偏移
- 通过libbpf处理结构体布局变化
- 依赖ELF节区中的类型信息

---

## 17. 实例分析

### 17.1 简单C程序

```c
// hello.c
#include <stdio.h>

const char* message = "Hello, ELF!";

int main() {
    printf("%s\n", message);
    return 0;
}
```

编译:
```bash
gcc -o hello hello.c
```

### 17.2 分析ELF结构

```bash
# 查看文件类型
$ file hello
hello: ELF 64-bit LSB executable, x86-64, ...

# 查看ELF头
$ readelf -h hello

# 查看程序头
$ readelf -l hello

# 查看节区
$ readelf -S hello

# 查看符号
$ readelf -s hello

# 查看动态链接
$ readelf -d hello
```

### 17.3 32位与64位对比

| 特性 | 32位 | 64位 |
|------|------|------|
| ELF头大小 | 52字节 | 64字节 |
| 程序头大小 | 32字节 | 56字节 |
| 节区头大小 | 40字节 | 64字节 |
| 地址宽度 | 32位 | 64位 |
| 魔术数偏移 | 0x100 | 0x200 (典型) |

### 11.4 创建和解析简单的ELF

```c
// 创建最简单的ELF (仅作示例)
#include <stdio.h>
#include <elf.h>

int main() {
    // ELF64头
    Elf64_Ehdr ehdr = {
        .e_ident = {
            0x7F, 'E', 'L', 'F', // 魔数
            ELFCLASS64,          // 64位
            ELFDATA2LSB,         // 小端
            EV_CURRENT,          // 版本
            ELFOSABI_NONE        // ABI
        },
        .e_type = ET_EXEC,
        .e_machine = EM_X86_64,
        .e_version = EV_CURRENT,
        .e_entry = 0x00400000,
        .e_phoff = sizeof(Elf64_Ehdr),
        .e_shoff = 0,
        .e_flags = 0,
        .e_ehsize = sizeof(Elf64_Ehdr),
        .e_phentsize = sizeof(Elf64_Phdr),
        .e_phnum = 0,
        .e_shentsize = sizeof(Elf64_Shdr),
        .e_shnum = 0,
        .e_shstrndx = 0
    };
    
    FILE *f = fopen("test.elf", "wb");
    fwrite(&ehdr, sizeof(ehdr), 1, f);
    fclose(f);
    
    return 0;
}
```

---

## 18. 工具与命令

### 18.1 readelf

```bash
# 查看ELF头
readelf -h <file>

# 查看程序头
readelf -l <file>

# 查看节区头
readelf -S <file>

# 查看符号表
readelf -s <file>

# 查看重定位
readelf -r <file>

# 查看动态段
readelf -d <file>

# 查看所有头信息
readelf -a <file>

# 解析动态链接
readelf --dyn-syms <file>
```

### 18.2 objdump

```bash
# 反汇编
objdump -d <file>

# 查看节区内容
objdump -s <file>

# 查看文件头
objdump -f <file>

# 显示符号表
objdump -t <file>

# C++filt 符号解码
echo "_Z4funcv" | c++filt
```

### 18.3 nm

```bash
# 查看所有符号
nm <file>

# 仅全局符号
nm -g <file>

# 仅动态符号
nm -D <file>

# 符号解码
nm -C <file>

# 按地址排序
nm -n <file>
```

### 18.4 ldd

```bash
# 查看动态库依赖
ldd <file>

# 显示依赖关系树
ldd -r <file>
```

### 18.5 patchelf

```bash
# 查看依赖
patchelf --print-interp <file>
patchelf --print-needed <file>
patchelf --print-rpath <file>

# 修改RPATH
patchelf --set-rpath /new/path <file>

# 添加依赖
patchelf --add-needed libfoo.so <file>

# 移除依赖
patchelf --remove-needed libfoo.so <file>
```

### 18.6 strip

```bash
# 移除符号表
strip <file>

# 移除指定节区
strip -R .comment <file>

# 仅复制文件(移除所有本地符号)
strip -s <file>
```

### 18.7 xxd / hexdump

```bash
# 十六进制查看
xxd <file>

# 只看头
xxd -l 64 <file>

# 小端反转显示
xxd -e <file>
```

---

## 19. 参考文献

### 官方规范

1. **ELF Specification**: https://refspecs.linuxfoundation.org/elf/elf.pdf
2. **System V Application Binary Interface**: https://refspecs.linuxfoundation.org/gabi/latest/
3. **AMD64 ABI**: https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.99.pdf
4. **ELF for ARM Architecture (ARM IHI 0044F)**: https://github.com/ARM-software/abi-aa
5. **ELF for RISC-V Architecture**: https://github.com/riscv-non-isa/riscv-elf-psabi-doc
6. **DWARF Debugging Standard**: https://dwarfstd.org/
7. **LoongArch ELF psABI**: https://loongson.github.io/LoongArch-ELF-ABI64/

### Linux文档

8. **Linux Man Pages**: man elf, man readelf, man objdump
9. **Linux Kernel Documentation - ELF**: Documentation/elf-notes.txt
10. **glibc Wiki - ELF**: https://sourceware.org/glibc/wiki/ELF
11. **Linux Kernel binfmt_elf.c**: https://github.com/torvalds/linux/blob/master/fs/binfmt_elf.c

### 工具文档

12. **binutils**: https://sourceware.org/binutils/docs/
13. **readelf**: https://man7.org/linux/man-pages/man1/readelf.1.html
14. **objdump**: https://man7.org/linux/man-pages/man1/objdump.1.html
15. **patchelf**: https://github.com/NixOS/patchelf
16. **LLVM ELF**: https://llvm.org/docs/ELF.html
17. **lld/LD**: https://lld.llvm.org/

### 书籍

18. **《Understanding the Linux Kernel》** - Daniel P. Bovet & Marco Cesati
19. **《Linkers and Loaders》** - John R. Levine
20. **《Programming Ground Up》** - Jonathan Bartlett
21. **《Linux Kernel Development》** - Robert Love
22. **《Computer Systems: A Programmer's Perspective》** - Randal E. Bryant & David R. O'Hallaron

### 安全相关

23. **Linux Security Wiki - RELRO**: https://gitlab.com/akiy/elf-loader/-/wikis/RELRO
24. **Intel CET Documentation**: https://www.intel.com/content/www/us/en/developer/articles/technical/technical-software-development-model-spec.html
25. **ARM TrustZone & ELF**: https://developer.arm.com/documentation/den0024/latest/
26. **Linux Kernel CET文档**: https://www.kernel.org/doc/html/next/x86/shstk.html
27. **RELRO深度解析**: https://redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro
28. **RELRO分析**: https://trapkit.de/articles/relro/

### 近年发展

29. **DWARF v5 Specification** (2017): 改进的调试信息压缩和索引
30. **DWARF v6** (2021): UTF-8改进支持
31. **ELF 4.3规范**: https://gabi.xinuos.com - 支持超过65000个节区
32. **RISC-V ELF psABI** (持续更新): https://github.com/riscv-non-isa/riscv-elf-psabi-doc
33. **Linux 6.x ELF相关更新**: 内核对新兴架构和安全特性的支持
34. **zstd压缩支持**: LLVM 15+/binutils 2.39+ 支持zstd压缩调试节区
35. **Compact Section Header Table**: ELF 4.3新特性，减少节区头开销
36. **RELR压缩**: https://maskray.me/blog/2021-10-31-relative-relocations-and-relr
37. **CREL提案**: https://discourse.llvm.org/t/rfc-crel-a-compact-relocation-format-for-elf/77600
38. **ELF进化 (MaskRay)**: https://maskray.me/blog/2024-05-26-evolution-of-elf-object-file-format

---

## 附录

### A. ELF 32位与64位类型对照

| 类型 (32) | 类型 (64) | 大小 (32/64) |
|-----------|-----------|--------------|
| Elf32_Addr | Elf64_Addr | 4/8 |
| Elf32_Half | Elf64_Half | 2/2 |
| Elf32_Off | Elf64_Off | 4/8 |
| Elf32_Sword | Elf64_Sxword | 4/8 |
| Elf32_Word | Elf64_Word | 4/8 |

### B. 常见节区及其用途

| 节区名 | 类型 | 用途 |
|--------|------|------|
| .text | PROGBITS | 可执行代码 |
| .data | PROGBITS | 已初始化全局/静态变量 |
| .bss | NOBITS | 未初始化全局/静态变量 |
| .rodata | PROGBITS | 只读数据 |
| .symtab | SYMTAB | 符号表 |
| .strtab | STRTAB | 字符串表 |
| .shstrtab | STRTAB | 节区名字符串表 |
| .dynsym | DYNSYM | 动态符号表 |
| .dynstr | STRTAB | 动态字符串表 |
| .plt | PROGBITS | 程序链接表 |
| .got | PROGBITS | 全局偏移表 |
| .got.plt | PROGBITS | PLT使用的GOT |
| .rela.dyn | RELA | 动态重定位 |
| .rela.plt | RELA | PLT重定位 |
| .dynamic | DYNAMIC | 动态链接信息 |
| .init | PROGBITS | 初始化代码 |
| .fini | PROGBITS | 结束代码 |
| .init_array | INIT_ARRAY | 构造函数数组 |
| .fini_array | FINI_ARRAY | 析构函数数组 |
| .tdata | PROGBITS | TLS初始化数据 |
| .tbss | NOBITS | TLS未初始化数据 |
| .debug_* | PROGBITS | DWARF调试信息 |
| .eh_frame | PROGBITS | 异常处理/栈展开 |
| .gnu.hash | GNU_HASH | GNU风格符号哈希表 |
| .gnu.version | VERSYM | 符号版本表 |
| .gnu.version_r | VERNEED | 版本需求 |
| .gnu.property | PROGBITS | GNU属性 |
| .comment | PROGBITS | 编译器版本信息 |
| .note.* | NOTE | 备注信息 |

### C. 动态标签快速参考

| 标签 | 值 | 说明 |
|------|-----|------|
| DT_NULL | 0 | 列表结束 |
| DT_NEEDED | 1 | 需要的共享库 |
| DT_HASH | 4 | 符号哈希表地址 |
| DT_STRTAB | 5 | 字符串表地址 |
| DT_SYMTAB | 6 | 符号表地址 |
| DT_RELA | 7 | 重定位表地址 |
| DT_RELASZ | 8 | 重定位表大小 |
| DT_STRSZ | 10 | 字符串表大小 |
| DT_SYMENT | 11 | 符号表项大小 |
| DT_INIT | 12 | 初始化函数 |
| DT_FINI | 13 | 结束函数 |
| DT_SONAME | 14 | 共享库名称 |
| DT_RPATH | 15 | 库搜索路径 (废弃) |
| DT_SYMBOLIC | 16 | 符号绑定优先 |
| DT_REL | 17 | REL类型重定位 |
| DT_RELSZ | 18 | REL大小 |
| DT_RELENT | 19 | REL项大小 |
| DT_PLTRELSZ | 20 | PLT重定位大小 |
| DT_PLTGOT | 21 | PLT/GOT地址 |
| DT_AUXILIARY | 32 | 辅助向量 |
| DT_FILTER | 33 | 过滤器 |
| DT_GNU_HASH | 0x6ffffef5 | GNU哈希表 |
| DT_TLSDESC_GOT | 0x6ffffef6 | TLS描述符GOT |
| DT_TLSDESC_PLT | 0x6ffffef7 | TLS描述符PLT |
| DT_RELRSZ | 35 | RELR重定位大小 |
| DT_RELR | 36 | RELR重定位表 |
| DT_RELRENT | 37 | RELR项大小 |
| DT_GNU_CONFLICT | 0x6ffffef8 | 冲突解决 |
| DT_GNU_LIBLIST | 0x6ffffef9 | 库列表 |
| DT_CONFIG | 0x6ffffefa | 配置 |
| DT_DEPAUDIT | 0x6ffffefb | 依赖审计 |
| DT_AUDIT | 0x6ffffefc | 审计 |
| DT_PLTPAD | 0x6ffffefd | PLT填充 |
| DT_MOVETAB | 0x6ffffefe | 移动表 |
| DT_SYMINFO | 0x6ffffeff | 符号信息 |
| DT_SYMTAB_SHNDX | 0x6fffffff | 扩展节区索引 |
| DT_VERSYM | 0x6ffffff0 | 符号版本 |
| DT_RELACOUNT | 0x6ffffff9 | RELA计数 |
| DT_RELCOUNT | 0x6ffffffa | REL计数 |
| DT_FLAGS_1 | 0x6ffffffb | 标志1 |
| DT_VERDEF | 0x6ffffffc | 版本定义 |
| DT_VERDEFNUM | 0x6ffffffd | 版本定义数 |
| DT_VERNEED | 0x6ffffffe | 版本需求 |
| DT_VERNEEDNUM | 0x6fffffff | 版本需求数 |
| DT_DEPRECATED_8BIT | 0x6000007d | 8位废弃 |
| DT_RPATH_32 | 0x6000007e | 32位RPATH |

### D. GNU_PROPERTY类型

| 属性 | 值 | 说明 |
|------|-----|------|
| GNU_PROPERTY_1 | 1 | 兼容属性 |
| GNU_PROPERTY_X86_1 | 0xc0000000 | x86处理器 |
| GNU_PROPERTY_X86_1_IBT | 1 | IBT (间接分支跟踪) |
| GNU_PROPERTY_X86_1_SHSTK | 2 | SHSTK (影子栈) |
| GNU_PROPERTY_STACK_SIZE | 3 | 栈大小提示 |
| GNU_PROPERTY_LOPROC | 0xc0000000 | 处理器特定起始 |
| GNU_PROPERTY_HIPROC | 0xdfffffff | 处理器特定结束 |
| GNU_PROPERTY_LOOS | 0xe0000000 | OS特定起始 |
| GNU_PROPERTY_HIOS | 0xefffffff | OS特定结束 |
| GNU_PROPERTY_ARM_1 | 0xc0000000 | ARM处理器 |
| GNU_PROPERTY_ARM_1_BTI | 1 | BTI (Branch Target Identification) |
| GNU_PROPERTY_ARM_1_PAC | 2 | PAC (Pointer Authentication) |

### E. 快速参考

```
魔数:        0x7F 'E' 'L' 'F'
32位类:      1
64位类:      2
小端:        1
大端:        2
ELF版本:     1 (EV_CURRENT)
```

### F. 常见安全编译选项

| 选项 | 效果 |
|------|------|
| -fPIE -pie | 位置无关可执行文件 |
| -fstack-protector | 栈保护 (经典) |
| -fstack-protector-strong | 栈保护 (强) |
| -fstack-protector-all | 栈保护 (全部) |
| -Wl,-z,relro,-z,now | Full RELRO |
| -Wl,-z,noexecstack | 无执行栈 |
| -D_FORTIFY_SOURCE=2 | FORTIFY_SOURCE |

### G. ELF文件分析清单

1. **文件类型识别**: file/xxd
2. **头部检查**: readelf -h
3. **程序头检查**: readelf -l
4. **节区检查**: readelf -S
5. **动态链接**: readelf -d
6. **符号表**: readelf -s / nm -C
7. **重定位**: readelf -r
8. **安全特性**: readelf -l (GNU_STACK) / -d (FLAGS)
9. **TLS**: readelf -S | grep -i tls
10. **DWARF**: readelf --debug-dump=info
11. **版本**: readelf -V
12. **哈希**: readelf --dyn-syms

---

*本文档由AI辅助生成，如有问题请反馈。*
