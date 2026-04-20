# ELF 程序头部与进程内存管理详解

## 目录

2. [程序头部 (Program Header)](#2-程序头部-program-header)
3. [GNU_PROPERTY 段详解](#3-gnu_property-段详解)
4. [进程虚拟地址空间](#4-进程虚拟地址空间)
5. [ELF 与进程内存的对应关系](#5-elf-与进程内存的对应关系)

---

## 2. 程序头部 (Program Header)

### 2.1 Elf64_Phdr 结构

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

### 2.2 段类型 (p_type)

| 类型值 | 名称 | 描述 |
|--------|------|------|
| PT_NULL | 0 | 未使用，忽略 |
| PT_LOAD | 1 | 可加载段 |
| PT_DYNAMIC | 2 | 动态链接信息 |
| PT_INTERP | 3 | 解释器路径 |
| PT_NOTE | 4 | 附加信息 |
| PT_PHDR | 6 | 程序头部表自身 |
| PT_TLS | 7 | 线程局部存储 |
| PT_GNU_EH_FRAME | 0x6474e550 | GCC 异常处理帧 |
| PT_GNU_STACK | 0x6474e551 | 堆栈标志 |
| PT_GNU_RELRO | 0x6474e552 | 只读重定位 |
| PT_GNU_PROPERTY | 0x6474e553 | GNU 属性 |

### 2.3 段标志 (p_flags)

| 标志值 | 名称 | 描述 |
|--------|------|------|
| PF_X (1) | 可执行 | 代码页可执行 |
| PF_W (2) | 可写 | 数据页可写 |
| PF_R (4) | 可读 | 数据页可读 |
| PF_MASKOS | 0x0ff00000 | OS 特定 |
| PF_MASKPROC | 0xf0000000 | 处理器特定 |

### 2.4 p_filesz 与 p_memsz 的差异

差异主要来自于 **SHT_NOBITS** 类型的节区（如 .bss）：

```bash
$ readelf -S /bin/ls | grep -E "(NOBITS|Filesz|Memsz)"

  [Nr] Name              Type          Size      FileSize  MemSize
  ...
  [16] .bss              NOBITS       0x01c000  0x00000   0x01c000  ← 不占文件空间
```

**结果**：`p_filesz < p_memsz`，差额就是 .bss 等节区的大小。

### 2.5 示例：/bin/ls 的 LOAD 段

```bash
$ readelf -l /bin/ls | grep LOAD

  LOAD   0x000000 0x0000000000000000 0x0000000000000000
         0x13a868 0x13a868  R E   0x200000   ← 代码+只读数据
  LOAD   0x13b000 0x000000000013b000 0x000000000013b000
         0x023a70 0x02e4e0  RW    0x200000   ← 可写数据+.bss
```

| 段 | p_filesz | p_memsz | 差值 | 说明 |
|----|----------|---------|------|------|
| 第1个 LOAD | 0x13a868 (1290280) | 0x13a868 | 0 | 代码+只读数据，无 .bss |
| 第2个 LOAD | 0x023a70 (146544) | 0x02e4e0 (190464) | **0xa970 (43408)** | 包含 .bss |

差值 **43408 字节** = `.bss` 中未初始化数据的大小。

---

## 3. GNU_PROPERTY 段详解

### 3.1 PT_GNU_PROPERTY 段概述

PT_GNU_PROPERTY (类型值 `0x6474e553`) 用于声明二进制文件的**硬件安全特性**。

### 3.2 字段含义

```
GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
               0x0000000000000030 0x0000000000000030  R      0x8
```

| 字段 | 值 | 含义 |
|------|-----|------|
| 段类型 | GNU_PROPERTY | GNU 属性段 |
| p_offset | 0x338 | 文件中段数据的偏移 |
| p_vaddr | 0x338 | 虚拟地址 |
| p_paddr | 0x338 | 物理地址（通常等于 vaddr） |
| p_filesz | 0x30 (48字节) | 文件中段大小 |
| p_memsz | 0x30 (48字节) | 内存中段大小 |
| p_flags | R | 标志：只读 (Read) |
| p_align | 0x8 | 对齐：8字节 |

### 3.3 常见 GNU_PROPERTY 属性类型

| 属性类型 | 值 | 描述 |
|----------|-----|------|
| GNU_PROPERTY_STACK_SIZE | 3 | 栈大小提示 |
| GNU_PROPERTY_X86_1 | 0xc0000000 | x86 处理器属性 |
| GNU_PROPERTY_X86_1_IBT | 1 | IBT (间接分支跟踪) |
| GNU_PROPERTY_X86_1_SHSTK | 2 | SHSTK (影子栈) |
| GNU_PROPERTY_ARM_1_BTI | 1 | BTI (Branch Target Identification) |
| GNU_PROPERTY_ARM_1_PAC | 2 | PAC (指针认证) |

### 3.4 查看 GNU_PROPERTY 内容

```bash
$ readelf -n /bin/ls

Displaying notes found in section .note.gnu.property:

  Owner                Data size        Description
  GNU                  0x00000020        NT_GNU_PROPERTY_TYPE_0
    Properties:        x86 feature: IBT, SHSTK
```

这表示启用了 **Intel CET (Control Flow Enforcement Technology)**：
- **IBT** = Indirect Branch Tracking：防止间接分支被劫持
- **SHSTK** = Shadow Stack：防止返回地址被篡改

### 3.5 与 GNU_STACK 的区别

| 段 | 作用 | p_flags |
|----|------|---------|
| **GNU_STACK** | 控制栈是否可执行 | RW（可读写不可执行） |
| **GNU_PROPERTY** | 声明安全特性 | R（只读） |

---

## 4. 进程虚拟地址空间

### 4.1 进程的组成

```
进程 (Process)
├── PCB (进程控制块)        ← 内核数据结构，不在进程虚拟地址空间中
├── 程序代码 (Program)       ← .text 节区 → 映射到虚拟地址空间
└── 相关数据段              ← 包括以下部分
    ├── .data 节区          ← 已初始化数据
    ├── .bss 节区           ← 未初始化数据
    ├── 堆 (Heap)           ← malloc/new 动态分配
    ├── 栈 (Stack)          ← 函数调用、局部变量
    └── 共享库映射          ← .so 文件映射
```

### 4.2 x86-64 Linux 用户空间布局

```
进程虚拟地址空间（x86-64 Linux 用户空间）
┌─────────────────────────────────────────────┐ 0xFFFFFFFFFFFFFFFF
│           内核空间 (kernel)                  │ ← 进程不可直接访问
├─────────────────────────────────────────────┤ 0xFFFF800000000000
│                                             │
│   共享库映射区域                             │ ← libc.so, ld-linux.so
│   (0x00007fff00000000 附近)                 │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│   堆 (Heap)                                │ ← malloc 分配 (向上增长)
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│   未初始化数据 (.bss)                        │
│   已初始化数据 (.data)                        │
│   代码段 (.text)                            │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│   加载基址区域                              │
│   (ET_EXEC: 固定 0x400000)                 │
│   (ET_DYN/PIE: 随机地址)                   │
│                                             │
└─────────────────────────────────────────────┘ 0x0000000000000000
```

### 4.3 关键概念

| 问题 | 答案 |
|------|------|
| PCB 属于虚拟地址空间吗？ | **否**。PCB 是内核数据结构，存在内核内存中 |
| 程序代码属于哪部分？ | 代码段 (.text)，属于虚拟地址空间 |
| 数据段属于哪部分？ | .data、.bss，属于虚拟地址空间 |
| **虚拟地址空间属于哪部分？** | **既不属于程序，也不属于数据，而是包含它们的"容器"** |

### 4.4 虚拟地址空间与物理内存的关系

| 概念 | 说明 |
|------|------|
| 虚拟地址空间 | 进程可寻址的地址范围（逻辑地址） |
| 物理内存 | 实际 RAM（物理地址） |
| 页表 | 虚拟地址到物理地址的映射（由内核管理） |

---

## 5. ELF 与进程内存的对应关系

### 5.1 ELF PT_LOAD 段到进程地址空间的映射

```bash
$ readelf -l /bin/ls | grep LOAD

  LOAD   0x000000 0x0000000000000000 0x0000000000000000
         0x13a868 0x13a868  R E   0x200000   → 映射到代码区域
  LOAD   0x13b000 0x000000000013b000 0x000000000013b000
         0x023a70 0x02e4e0  RW    0x200000   → 映射到数据区域
```

| ELF 内容 | 映射到虚拟地址空间 |
|----------|-------------------|
| .text, .rodata | 对应 vaddr 范围 |
| .data | 对应 vaddr 范围 |
| .bss | 只占 p_memsz，不占 p_filesz |

### 5.2 进程内存指标与 ELF 的关系

```bash
$ ps aux | head -1
USER       PID %CPU %MEM    VSZ   RSS TTY
root     12345  0.0  0.1  123456  7890 ...
```

| 指标 | ELF 对应 | 含义 |
|------|---------|------|
| VSZ | ≈ p_memsz 总和 | 进程虚拟地址空间大小 |
| RSS | 实际分配 | 实际占用的物理内存 |

### 5.3 p_filesz / p_memsz 与系统内存概念的关系

| 字段 | 含义 | 与系统内存的关系 |
|------|------|-----------------|
| p_filesz | 文件中实际大小 | 磁盘文件大小 |
| p_memsz | 内存中需要的大小 | 进程虚拟地址空间的一部分 |

**关键差异**：.bss 节区占 p_memsz 但不占 p_filesz

### 5.4 总结表

| ELF 概念 | 操作系统概念 | 说明 |
|----------|-------------|------|
| p_filesz | 磁盘文件大小 | 文件占用空间 |
| p_memsz | 进程虚拟地址空间 | 运行时内存需求 |
| PT_LOAD 映射 | 进程地址空间区域 | 代码、数据等 |
| PCB | 内核内存 | 不在进程地址空间内 |
| 堆/栈 | 进程地址空间 | 动态分配的区域 |

---

## 附录：常见误解澄清

### 误解 1：DYN = 动态库

**事实**：DYN 只表示"使用动态库加载机制"，可能是：
- 真正的 .so 文件
- PIE 可执行文件

### 误解 2：PIE 不可执行

**事实**：PIE **就是可执行文件**，只是使用了位置无关的加载方式。

### 误解 3：p_filesz 等于进程内存大小

**事实**：p_memsz 只是进程地址空间的一部分，还有堆、栈、共享内存等。

---

*本文档由 AI 辅助生成*
