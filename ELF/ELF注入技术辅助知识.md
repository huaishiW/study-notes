# ELF 注入技术知识总结

本文档系统整理了 ELF 文件注入技术所涉及的核心知识领域，包括 ELF 段结构、x86-64 机器码、地址跳转、文件修改、内存映射以及进程注入原理。

---

## 一、ELF PT_LOAD 段结构

### 1.1 Program Header 结构体定义 (ELF-64)

```c
typedef struct {
    uint32_t   p_type;     // 段类型
    uint32_t   p_flags;    // 段权限标志
    Elf64_Off   p_offset;   // 段在文件中的偏移
    Elf64_Addr  p_vaddr;    // 段的虚拟地址
    Elf64_Addr  p_paddr;    // 段的物理地址 (通常与 p_vaddr 相同)
    uint64_t   p_filesz;    // 段在文件中的大小
    uint64_t   p_memsz;    // 段在内存中的大小
    uint64_t   p_align;    // 段对齐要求
} Elf64_Phdr;
```

### 1.2 常见段类型

| 类型值 | 名称 | 说明 |
|--------|------|------|
| `PT_NULL` | 0 | 未使用的占位符 |
| `PT_LOAD` | 1 | **可加载段** — 最重要的类型，用于将文件内容映射到内存 |
| `PT_DYNAMIC` | 2 | 动态链接信息 |
| `PT_INTERP` | 3 | 解释器路径名 (如 `/lib64/ld-linux-x86-64.so.2`) |
| `PT_NOTE` | 4 | 辅助信息/元数据 |
| `PT_PHDR` | 6 | 程序头表自身的位置 |
| `PT_TLS` | 7 | 线程局部存储 |
| `PT_GNU_STACK` | 0x6474e551 | 栈的可执行属性 |

### 1.3 段权限标志 (p_flags)

权限由 3 个 bit 位组成，组合方式如下：

| bit 位 | 值 | 含义 |
|--------|-----|------|
| `PF_READ` | 0x4 | 可读 (R) |
| `PF_WRITE` | 0x2 | 可写 (W) |
| `PF_EXEC` | 0x1 | 可执行 (X) |

常见权限组合：

| 标志值 | 含义 |
|--------|------|
| `R` | 0x4 — 只读数据段 |
| `RX` | 0x5 — 只读代码段 |
| `RW` | 0x6 — 可读写数据段 |
| `RWX` | 0x7 — 可读写执行段 |

### 1.4 PT_LOAD 段的加载原理

**核心规则**：`p_vaddr` 和 `p_offset` 必须对 `p_align` (通常为页大小 0x1000 = 4096) 取模相等。

```
(p_vaddr - p_offset) % p_align == 0
```

这确保操作系统在加载段时可以通过简单的加减运算得到内存映射地址。

**文件与内存的映射关系**：

```
文件偏移范围:  [p_offset, p_offset + p_filesz)
内存地址范围:  [p_vaddr, p_vaddr + max(p_filesz, p_memsz))
```

如果 `p_memsz > p_filesz`，超出部分在内存中被初始化为 0 (用于 .bss)。

### 1.5 段与 Section 的区别

| 特性 | Segment (Program Header) | Section (Section Header) |
|------|------------------------|--------------------------|
| 用途 | **运行时** — 供操作系统加载使用 | **链接时** — 供链接器使用 |
| 是否加载到内存 | 由 PT_LOAD 段决定 | 否 (section headers 不参与加载) |
| 包含内容 | 连续的内存映射数据 | 链接器需要的元数据 |
| 在内存中可见 | 是 | 否 |

### 1.6 典型 ELF 文件布局

```
文件布局:
┌────────────────────────────────────────────┐
│  ELF Header (e_shoff 指向节头表)            │
│  Program Header Table (e_phoff 指向段头表)  │
│  Segment 1: .text (RX) ──────────────┐     │
│  Segment 2: .data (RW) ──────────────┼─    │ ← 这两个段之间可能有间隙
│  ...                                    │     │
│  Section Headers                         │     │
└────────────────────────────────────────────┘

内存布局 (假设基址 0x400000):
┌────────────────────────────────────────────┐
│  0x400000: ELF Header                       │
│  0x400040: Program Headers                  │
│  0x401000: .text segment [p_vaddr=0x401000] │
│  0x40xxxx: (padding) ← CODE CAVE           │ ← 注入点
│  0x40yyyy: .data segment [p_vaddr=0x40yyyy]│
└────────────────────────────────────────────┘
```

---

## 二、x86-64 机器码与调用约定

### 2.1 系统调用约定 (Linux x86-64)

Linux 系统调用使用 `syscall` 指令，参数传递遵循以下约定：

| 参数序号 | 寄存器 | 用途 |
|---------|--------|------|
| 第1个参数 | `rdi` | (系统调用号对应的第1个参数) |
| 第2个参数 | `rsi` | 第2个参数 |
| 第3个参数 | `rdx` | 第3个参数 |
| 第4个参数 | `r10` | 第4个参数 (注意：不是 rcx) |
| 第5个参数 | `r8` | 第5个参数 |
| 第6个参数 | `r9` | 第6个参数 |
| 返回值 | `rax` | syscall 返回值 |

**常见系统调用号 (x86-64)**:

| 调用号 | 名称 | 用途 |
|--------|------|------|
| `0` | `read` | 读取文件/文件描述符 |
| `1` | `write` | 写入文件/文件描述符 |
| `60` | `exit` | 退出进程 |
| `231` | `exit_group` | 退出所有线程 |

### 2.2 常用 x86-64 机器码对照表

#### 数据传输指令

| 指令 | 机器码 (字节) | 说明 |
|------|--------------|------|
| `push rax` | `50` | 将 rax 入栈 |
| `push rbx` | `53` | 将 rbx 入栈 |
| `push rdi` | `57` | 将 rdi 入栈 |
| `push rsi` | `56` | 将 rsi 入栈 |
| `push rdx` | `52` | 将 rdx 入栈 |
| `push rbp` | `55` | 将 rbp 入栈 |
| `push r8` | `41 50` | 将 r8 入栈 (需要 REX 前缀 0x41) |
| `push r9` | `41 51` | 将 r9 入栈 |
| `push r10` | `41 52` | 将 r10 入栈 |
| `push r11` | `41 53` | 将 r11 入栈 |
| `push r12` | `41 54` | 将 r12 入栈 |
| `push r13` | `41 55` | 将 r13 入栈 |
| `push r14` | `41 56` | 将 r14 入栈 |
| `push r15` | `41 57` | 将 r15 入栈 |
| `pop rax` | `58` | 从栈弹出到 rax |
| `pop r8` | `41 58` | 从栈弹出到 r8 |
| `pop r15` | `41 5f` | 从栈弹出到 r15 |
| `pop rbx` | `5b` | 从栈弹出到 rbx |
| `pop rbp` | `5d` | 从栈弹出到 rbp |
| `pop rdi` | `5f` | 从栈弹出到 rdi |
| `pop rsi` | `5e` | 从栈弹出到 rsi |
| `pop rdx` | `5a` | 从栈弹出到 rdx |
| `pop rcx` | `59` | 从栈弹出到 rcx |
| `pop r8` | `41 58` | 从栈弹出到 r8 |
| `pop r9` | `41 59` | 从栈弹出到 r9 |

#### 移动指令

| 指令 | 机器码 | 说明 |
|------|--------|------|
| `mov eax, 0x1` | `b8 01 00 00 00` | 将立即数 1 移动到 eax (小立即数) |
| `mov edi, 0x1` | `bf 01 00 00 00` | 将立即数 1 移动到 edi |
| `mov edx, 0xb` | `ba 0b 00 00 00` | 将立即数 11 移动到 edx |
| `mov rsi, rax` | `48 89 c6` | REX.W + 89 + modrm(rsi, rax) |
| `nop` | `90` | 无操作指令 |

#### 控制流指令

| 指令 | 机器码 | 说明 |
|------|--------|------|
| `jmp short $+2` | `eb xx` | 相对短跳转，xx 是跳转位移 |
| `jmp rel32` | `e9 xx xx xx xx` | 相对近跳转，32位位移 |
| `call rel32` | `e8 xx xx xx xx` | 相对调用，32位位移 |
| `syscall` | `0f 05` | 系统调用 |
| `ret` | `c3` | 函数返回 |

#### REX 前缀说明

x86-64 指令集中，当需要使用 r8-r15、rsp 等扩展寄存器时，需要使用 `REX` 前缀 (`0x40-0x4F`)：

| REX 前缀 | 值 | 说明 |
|----------|-----|------|
| `REX.B` | +1 | 扩展基址寄存器 |
| `REX.X` | +2 | 扩展索引寄存器 |
| `REX.R` | +4 | 扩展 ModR/M 寄存器 |
| `REX.W` | +8 | 64位操作数大小 |

```
0x41 = 0100_0001
  ││││└──── W=0 (32位操作数)
  │││└─── R=0 (非扩展寄存器)
  ││└─── X=0 (非扩展索引)
  │└─── B=1 (使用扩展寄存器 r8-r15)
  └───── 固定为1
```

### 2.3 wrapper.c 中 shellcode 的机器码详解

```nasm
; wrapper.c 中的 shellcode 逐字节解析
; ==============================================

; 第1部分: 保存所有寄存器 (23 字节)
50,                          ; push rax
53,                          ; push rbx
51,                          ; push rcx
52,                          ; push rdx
56,                          ; push rsi
57,                          ; push rdi
55,                          ; push rbp
41, 50,                      ; push r8
41, 51,                      ; push r9
41, 52,                      ; push r10
41, 53,                      ; push r11
41, 54,                      ; push r12
41, 55,                      ; push r13
41, 56,                      ; push r14
41, 57,                      ; push r15

; 第2部分: jmp short 跳过数据区
eb, 2f,                      ; jmp $+0x2f (跳转 47 字节)

; 第3部分: pop 指令获取字符串地址
5e,                          ; pop rsi (获取字符串 "helloworld\n" 的地址)

; 第4部分: write(1, rsi, 11)
bf, 01, 00, 00, 00,         ; mov edi, 1        (文件描述符 stdout)
ba, 0b, 00, 00, 00,         ; mov edx, 11      (字符串长度)
b8, 01, 00, 00, 00,         ; mov eax, 1        (sys_write 调用号)
0f, 05,                      ; syscall

; 第5部分: 恢复所有寄存器 (23 字节)
41, 5f,                      ; pop r15
41, 5e,                      ; pop r14
41, 5d,                      ; pop r13
41, 5c,                      ; pop r12
41, 5b,                      ; pop r11
41, 5a,                      ; pop r10
41, 59,                      ; pop r9
41, 58,                      ; pop r8
5d,                          ; pop rbp
5f,                          ; pop rdi
5e,                          ; pop rsi
5a,                          ; pop rdx
59,                          ; pop rcx
5b,                          ; pop rbx
58,                          ; pop rax

; 第6部分: 跳转到原入口点 (相对跳转)
e9, 00, 00, 00, 00,         ; jmp rel32 (待填入的立即数)

; 第7部分: 填充
90,                          ; nop

; 第8部分: call 指令 (回到字符串处)
e8, cc, ff, ff, ff,         ; call rel32(-52) → 执行后压栈返回地址

; 第9部分: 数据
'h','e','l','l','o','w','o','r','l','d','\n'
```

---

## 三、R_X86_64_PC32 相对地址跳转

### 3.1 什么是 PC-relative 寻址

x86-64 架构的一个核心特性是 **PC-relative (RIP-relative) 寻址**。大多数跳转和调用指令使用相对地址，而不是绝对地址。

这意味着指令中编码的不是目标内存地址，而是一个**偏移量**：

```
跳转后下一条指令的地址 + 偏移量 = 目标地址
```

### 3.2 相对跳转指令编码

#### JMP rel32 (Near Jump, Relative)

```
机器码:  e9  xx xx xx xx
         └── 4 字节的相对偏移量 (小端序)
```

**计算公式**:
```
rel32 = 目标地址 - (当前指令地址 + 5)
```

其中 5 是 `jmp rel32` 指令本身的字节长度。

#### JMP rel8 (Short Jump)

```
机器码:  eb  xx
         └── 1 字节的相对偏移量
```

**计算公式**:
```
rel8 = 目标地址 - (当前指令地址 + 2)
```

#### CALL rel32

```
机器码:  e8  xx xx xx xx
         └── 4 字节的相对偏移量
```

**计算公式**:
```
rel32 = 目标地址 - (当前指令地址 + 5)
```

执行 `call` 时，CPU 会自动将返回地址 (call 指令后的下一条指令地址) 压栈。

### 3.3 wrapper.c 中的跳转计算

```c
// wrapper.c 第162行的计算
insert_vaddr  // shellcode 在内存中的起始虚拟地址
orig_entry    // 原程序的入口点虚拟地址

// jmp 指令位于 shellcode 偏移 0x67 处，指令长度 5 字节
// 所以跳转到下一条指令的地址是 insert_vaddr + 67 + 5 = insert_vaddr + 71

int32_t entry_rel = (int32_t)(orig_entry - (insert_vaddr + 71));
//                                                        ↑
//                                              jmp 指令执行时的 RIP

// 填入 jmp 指令的立即数字段 (offset 67, 4 bytes)
memcpy(shellcode + 67, &entry_rel, 4);
```

**图示**:
```
          insert_vaddr                    insert_vaddr + 71
               │                                 │
               ▼                                 ▼
    ┌─────────────────────┐           ┌─────────────────────┐
    │      shellcode      │           │                     │
    │  ...  [67] [68-71]  │           │                     │
    │          ↑          │           │                     │
    │     e9 ????         │           │                     │
    └─────────────────────┘           │                     │
                    │                 │                     │
                    │    ┌────────────┘                     │
                    │    │                                  │
                    ▼    ▼                                  ▼
                 ┌──────────────────────────────────────────────┐
                 │  jmp rel32 = orig_entry - (insert_vaddr+71)  │
                 └──────────────────────────────────────────────┘
                                    │
                                    │
                                    ▼
                             orig_entry
                           (原程序入口点)
```

### 3.4 PIE/ASLR 下的跳转

由于 wrapper.c 计算的是 **虚拟地址差值** 而非文件偏移，所以它**兼容 PIE (Position Independent Executable)** 和 **ASLR**：

- `orig_entry` 是从被感染 ELF 的 Header 中直接读取的真实虚拟地址
- `insert_vaddr` 是通过段头信息计算出的真实虚拟地址
- 两者都基于同一个加载基址计算，因此差值是准确的

### 3.5 栈布局与 call/pop 技巧

wrapper.c 使用了一个经典的 **call/pop 技巧** 来获取字符串字面量的地址：

```nasm
; 代码序列:
eb, 2f,     ; jmp $+0x2f  (跳过字符串数据)
5e,         ; pop rsi     (获取字符串地址)

; ... 中间是字符串数据 ...
; 'h','e','l','l','o','w','o','r','l','d','\n'
; ... 和 call 指令 ...
e8, cc, ff, ff, ff, ; call rel32(-52)  → 跳回 pop rsi 的下一条指令
```

**原理**:
1. `call` 指令会将下一条指令的地址 (即字符串数据的地址) 压入栈中
2. 然后执行 `pop rsi` 将该地址弹出到 rsi 寄存器
3. 之后再执行 `ret` (由 `call` 自动压栈的返回地址) 跳回原流程

```
执行流程:
  jmp short +0x2f
       │
       │  (跳过字符串数据)
       ▼
  pop rsi          ←──┐
       │               │
  [字符串 "helloworld\n"]  │
       │               │
  call -52 ──────────┘  (call 把这里压栈，然后跳回 pop rsi)
```

---

## 四、ELF 文件结构修改 (p_filesz / p_memsz)

### 4.1 p_filesz 与 p_memsz 的区别

| 字段 | 含义 | 用途 |
|------|------|------|
| `p_filesz` | 段在**文件**中的实际字节数 | 告诉操作系统从 ELF 文件中读取多少数据 |
| `p_memsz` | 段在**内存**中的总字节数 | 告诉操作系统分配多少内存，超出部分清零 |

通常 `p_filesz <= p_memsz`。当 `p_memsz > p_filesz` 时：
- 操作系统从文件加载 `p_filesz` 字节
- 剩余 `(p_memsz - p_filesz)` 字节在内存中被初始化为 0
- 这就是 .bss 段的工作原理

### 4.2 注入时为什么要同时修改两个字段

```c
// wrapper.c 第194-199行
for (int i = 0; i < phnum; i++) {
    if (phdr[i].p_type == PT_LOAD && phdr[i].p_vaddr == target_vaddr) {
        phdr[i].p_filesz += SHELLCODE_SIZE;   // 文件大小也要扩展
        phdr[i].p_memsz  += SHELLCODE_SIZE;    // 内存大小也要扩展
        break;
    }
}
```

**为什么两者都需要？**

| 修改 | 不修改的后果 |
|------|-------------|
| 只修改 `p_filesz` | 文件末尾的数据不会被加载到内存 |
| 只修改 `p_memsz` | 文件中没有对应数据，内存中会是未初始化的垃圾值 |

### 4.3 扩展文件的步骤

```c
// 1. 如果 shellcode 超出原文件大小，需要先扩展文件
new_file_size = insert_off + SHELLCODE_SIZE;
if (new_file_size > st.st_size) {
    ftruncate(fd, new_file_size);  // 扩展文件到新大小

    // 重新映射，因为内存区域大小变了
    munmap(file_map, st.st_size);
    file_map = mmap(NULL, new_file_size,
                    PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
}

// 2. 写入 shellcode
memcpy(file_map + insert_off, shellcode, SHELLCODE_SIZE);

// 3. 扩展段的两个大小字段
phdr[i].p_filesz += SHELLCODE_SIZE;
phdr[i].p_memsz  += SHELLCODE_SIZE;

// 4. 修改入口点
ehdr->e_entry = insert_vaddr;

// 5. 强制同步到磁盘
msync(file_map, new_file_size, MS_SYNC);
```

### 4.4 为什么不修改 p_align 的段也能工作

假设原段 `p_align = 0x1000` (4KB 页面对齐)，修改 `p_filesz` 后：

```
原段: p_offset=0x1000, p_vaddr=0x401000, p_filesz=0x5000, p_align=0x1000
感染后: p_filesz=0x5080

检查: p_vaddr (0x401000) % p_align (0x1000) = 0 ✓
     p_offset (0x1000) % p_align (0x1000) = 0 ✓

只要这两个条件满足，加载器就能正常工作。
```

关键在于：**只有前两个 LOAD 段需要满足对齐约束**，感染后我们只改变了段大小，没有改变段的起始地址 (`p_offset` 和 `p_vaddr`)，所以对齐条件仍然满足。

---

## 五、内存映射 (mmap) 与直接文件操作

### 5.1 mmap 的基本原理

`mmap()` 将文件或设备映射到进程的虚拟地址空间。写入映射区域等同于直接写入文件。

```c
void *mmap(void *addr,           // 希望映射到的地址，NULL 表示由内核选择
           size_t length,       // 映射长度
           int prot,            // 内存保护权限
           int flags,           // 映射类型标志
           int fd,              // 文件描述符
           off_t offset);       // 文件中的偏移量
```

### 5.2 PROT_* 权限标志

| 标志 | 含义 |
|------|------|
| `PROT_READ` | 映射区域可读 |
| `PROT_WRITE` | 映射区域可写 |
| `PROT_EXEC` | 映射区域可执行 |
| `PROT_NONE` | 映射区域不可访问 |

### 5.3 MAP_* 类型标志

| 标志 | 含义 |
|------|------|
| `MAP_SHARED` | 对映射区域的修改**直接写入文件** (写透) |
| `MAP_PRIVATE` | 对映射区域的修改**不写入文件** (COW 副本) |
| `MAP_ANONYMOUS` | 不绑定任何文件 (匿名映射，用于分配内存) |

### 5.4 wrapper.c 中的 mmap 用法

```c
// 打开目标文件
int fd = open(target, O_RDWR);    // 必须 O_RDWR 才能 mmap PROT_WRITE

// 将文件映射到内存，PROT_READ | PROT_WRITE 允许读写
// MAP_SHARED 意味着对映射区域的写入会直接反映到文件
unsigned char *file_map = mmap(
    NULL,                           // addr = NULL，由内核选择地址
    st.st_size,                     // 映射整个文件
    PROT_READ | PROT_WRITE,         // 可读写
    MAP_SHARED,                     // 写入直接同步到文件
    fd,                             // 文件描述符
    0                               // offset = 0，从文件开头开始
);
```

**MAP_SHARED vs MAP_PRIVATE**：

```
MAP_SHARED:
  进程写入映射区域 ──→ 内核直接写入磁盘文件
                    另一个进程映射同一文件也能看到修改

MAP_PRIVATE:
  进程写入映射区域 ──→ 内核创建副本(Copy-on-Write)
                    磁盘文件不变，只有本进程看到修改
```

### 5.5 msync 强制同步

```c
msync(file_map, new_file_size, MS_SYNC);
// 参数: MS_SYNC  同步等待写入完成
//       MS_ASYNC 异步写入，不等待
```

使用 `MAP_SHARED` 时，内核会延迟将修改写回磁盘。调用 `msync(MS_SYNC)` 确保数据立即持久化到磁盘。

### 5.6 ftruncate 扩展文件

```c
#include <sys/fcntl.h>
ftruncate(fd, new_file_size);
```

`ftruncate` 将文件扩展到指定大小。如果文件比这个值小，会用零填充扩展。

---

## 六、进程注入与代码注入原理

### 6.1 代码注入的两大类别

#### 静态注入 (Static Injection / Binary Infection)

将代码**永久地**写入目标 ELF 文件，修改磁盘上的二进制文件。

| 特点 | 说明 |
|------|------|
| 修改对象 | ELF 文件本身 (磁盘上的文件) |
| 持久性 | 永久生效，重启后仍有效 |
| 触发时机 | 下次执行被感染的程序时 |
| 示例 | wrapper.c、elf_injector |

#### 动态注入 (Dynamic Injection / Process Injection)

在**进程运行时**将代码注入到其内存空间中，不修改磁盘文件。

| 特点 | 说明 |
|------|------|
| 修改对象 | 进程的内存空间 |
| 持久性 | 仅在当前进程生命周期内有效 |
| 触发时机 | 注入后立即执行 |
| 示例 | ptrace 注入、/proc/pid/mem 注入 |

### 6.2 静态注入的工作原理

wrapper.c 的静态注入流程：

```
┌─────────────────────────────────────────────────────────┐
│                   注入前: 干净 ELF                        │
├─────────────────────────────────────────────────────────┤
│  ELF Header: e_entry = 0x401000                         │
│  Program Header:                                        │
│    LOAD [RX] offset=0x1000, vaddr=0x401000              │
│            filesz=0x5000                                │
│                                                         │
│  [ELF Header][Phdr][.text 代码...][.data][Section Hdrs]  │
└─────────────────────────────────────────────────────────┘
                           │
                    wrapper.c 写入 shellcode
                    修改 e_entry 和 p_filesz/p_memsz
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   注入后: 被感染 ELF                      │
├─────────────────────────────────────────────────────────┤
│  ELF Header: e_entry = 0x401500  ← 指向 code cave        │
│  Program Header:                                        │
│    LOAD [RWX] offset=0x1000, vaddr=0x401000             │
│            filesz=0x5080  ← 扩展了 80 字节               │
│                                                         │
│  [ELF Header][Phdr][.text 代码...][shellcode][Section]   │
│                                    ↑                    │
│                               新入口点 0x401500          │
└─────────────────────────────────────────────────────────┘
                           │
                    执行被感染的程序
                           ▼
┌─────────────────────────────────────────────────────────┐
│  1. 内核加载 ELF，读取 e_entry = 0x401500                 │
│  2. CPU 从 0x401500 开始执行 → shellcode                  │
│  3. shellcode: write(1, "helloworld\n", 11)             │
│  4. jmp rel32 → 跳转回 0x401000 (原入口)                  │
│  5. 继续执行原始程序代码 (完全不受影响)                      │
└─────────────────────────────────────────────────────────┘
```

### 6.3 入口点重定向的原理

`e_entry` 字段告诉内核 **程序的第一条指令应该从哪里开始执行**。

```c
// 修改入口点
ehdr->e_entry = insert_vaddr;  // 指向 shellcode 的虚拟地址
```

内核在创建进程时的简化步骤：

```c
// 内核加载 ELF 时的伪代码
struct elf_binary {
    Elf64_Ehdr *header;
    Elf64_Phdr *program_headers;
};

void load_elf(struct elf_binary *bin) {
    // 1. 解析 ELF Header
    Elf64_Ehdr *ehdr = bin->header;

    // 2. 将各个 PT_LOAD 段映射到内存
    for (each PT_LOAD segment) {
        mmap(segment->p_vaddr, segment->p_memsz,
             segment->p_flags,
             MAP_SHARED, fd, segment->p_offset);
    }

    // 3. 设置初始指令指针 = e_entry
    set_instruction_pointer(ehdr->e_entry);  // ← 关键：这里是 shellcode 的地址

    // 4. 开始执行
    return_to_user_mode();
}
```

### 6.4 Code Cave 的本质

Code Cave 是段末尾与下一个段起始之间的**文件偏移间隙**：

```
文件布局:
┌────────────────────────────────────────────────────┐
│  0x1000: 可执行 LOAD 段开始                          │
│          ... 原有代码 ...                           │
│  0x5000: 段结束 (p_filesz)  ← 我们的注入点            │
│          [GAP / CODE CAVE]  ← 空闲空间              │
│  0x5080: 下一个 LOAD 段开始                          │
└────────────────────────────────────────────────────┘
```

这个间隙在内存中：
- 操作系统按 `p_align` (通常是页大小 0x1000) 对齐每个段
- 段的内存大小为 `p_memsz`，可能包含这段未映射的"间隙"
- 或者该区域是段的一部分但未被使用（填充字节为 0）

### 6.5 Shellcode 执行的完整流程

```
1. 用户执行 ./infected_program
          │
          ▼
2. Shell 调用 execve() 系统调用
          │
          ▼
3. 内核解析 ELF Header，读取 e_entry = 0x401500
          │
          ▼
4. 内核创建进程内存空间，按 PT_LOAD 映射各个段
          │  (p_filesz/p_memsz 已被修改，所以 shellcode 也被映射)
          ▼
5. 设置 CPU 的指令指针 (RIP) = 0x401500
          │
          ▼
6. CPU 开始执行 shellcode:
   ┌────────────────────────────────────────────┐
   │ 0x401500: push rax / rbx / ... (保存寄存器)  │
   │ 0x401507: jmp short +0x2f                  │
   │ 0x401509: pop rsi                          │
   │ 0x40150a: mov edi, 1                       │
   │ 0x40150f: mov edx, 11                      │
   │ 0x401514: mov eax, 1                       │
   │ 0x401519: syscall  → 输出 "helloworld\n"    │
   │ 0x40151b: pop r15 / r14 / ... (恢复寄存器)   │
   │ 0x40152c: jmp rel32 → orig_entry           │
   └────────────────────────────────────────────┘
          │
          ▼
7. jmp 指令跳转到原入口点 0x401000
          │
          ▼
8. 继续执行原始程序，完全无感知
          │
          ▼
9. 程序正常退出
```

### 6.6 为什么 shellcode 执行后原程序不受影响

关键在于：

1. **Shellcode 插入在 code cave** — 这是段的末尾填充区域，不包含任何原有代码
2. **寄存器完全恢复** — `push` 保存的所有寄存器在 `pop` 后完全恢复
3. **跳转回原入口** — `jmp rel32` 使执行流完全回到原始代码
4. **段大小只增不减** — 原有的代码数据没有任何改变

### 6.7 动态注入简介 (对比静态注入)

| 维度 | 静态注入 (wrapper.c) | 动态注入 (ptrace/etc.) |
|------|---------------------|----------------------|
| 目标文件 | ELF 二进制文件 (磁盘) | 正在运行的进程 (内存) |
| 修改持久性 | 永久 (文件被修改) | 临时 (仅内存有效) |
| 需要权限 | 写权限 (打开文件) | 调试权限 (ptrace/dBG) |
| 复杂度 | 中等 | 较高 (需要进程间通信) |
| 检测难度 | 较低 (文件有改动) | 较高 (内存注入) |
| 典型工具 | wrapper.c, elf_injector | gdb, strace, Cymothoa |

---

## 七、综合实例：wrapper.c 注入算法全流程

```
┌────────────────────────────────────────────────────────────────┐
│                    wrapper.c 注入完整流程                         │
└────────────────────────────────────────────────────────────────┘

1. 文件操作阶段
   ┌────────────────────────────────────────────────────────────┐
   │  open(target, O_RDWR)          打开目标 ELF (读写)          │
   │  fstat(fd, &st)                获取文件大小                 │
   │  mmap(..., MAP_SHARED)         映射到内存                   │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
2. ELF 解析阶段
   ┌────────────────────────────────────────────────────────────┐
   │  ehdr = file_map                               ELF Header   │
   │  phdr = file_map + ehdr->e_phoff              Program Headers │
   │  验证 ELF 魔数 (0x7f, 'E', 'L', 'F')           验证格式    │
   │  qsort(phdr, phdr_cmp)                        按虚拟地址排序 │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
3. 查找可注入位置
   ┌────────────────────────────────────────────────────────────┐
   │  遍历找到: PT_LOAD 且 PF_X 的段                            │
   │  target_vaddr = phdr[i].p_vaddr                            │
   │  target_file_off = phdr[i].p_offset                        │
   │  target_file_end = target_file_off + p_filesz              │
   │  寻找下一个 LOAD 段的 p_offset                             │
   │  gap = next_load_offset - target_file_end                 │
   │  if (gap < SHELLCODE_SIZE) → 空间不足，退出                │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
4. 计算注入参数
   ┌────────────────────────────────────────────────────────────┐
   │  insert_off = target_file_end              注入文件偏移     │
   │  insert_vaddr = target_vaddr +            注入虚拟地址      │
   │      (insert_off - target_file_off)                           │
   │  orig_entry = ehdr->e_entry                原始入口点     │
   │  计算 jmp 相对位移:                                     │
   │    entry_rel = orig_entry - (insert_vaddr + 71)            │
   │  填入 shellcode[67..70]                                   │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
5. 文件扩展 (如果需要)
   ┌────────────────────────────────────────────────────────────┐
   │  new_size = insert_off + SHELLCODE_SIZE                   │
   │  if (new_size > old_size) {                               │
   │      ftruncate(fd, new_size);   扩展文件                  │
   │      munmap + remmap;           重新映射                   │
   │  }                                                        │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
6. 写入 shellcode
   ┌────────────────────────────────────────────────────────────┐
   │  memcpy(file_map + insert_off, shellcode, SHELLCODE_SIZE)  │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
7. 修改 ELF 结构
   ┌────────────────────────────────────────────────────────────┐
   │  phdr[i].p_filesz += SHELLCODE_SIZE   扩展文件大小         │
   │  phdr[i].p_memsz  += SHELLCODE_SIZE   扩展内存大小         │
   │  ehdr->e_entry = insert_vaddr         新入口点             │
   └────────────────────────────────────────────────────────────┘
                              │
                              ▼
8. 持久化并清理
   ┌────────────────────────────────────────────────────────────┐
   │  msync(MAP_SHARED, MS_SYNC)        强制写回磁盘            │
   │  munmap                             解除映射               │
   │  close(fd)                          关闭文件               │
   └────────────────────────────────────────────────────────────┘
```

---

## 八、关键知识点速查表

### 8.1 ELF 核心字段速查

| 字段 | 在哪 | 用途 |
|------|------|------|
| `e_entry` | ELF Header | 程序入口点的虚拟地址 |
| `e_phoff` | ELF Header | Program Header Table 的文件偏移 |
| `e_shoff` | ELF Header | Section Header Table 的文件偏移 |
| `e_phentsize` | ELF Header | 每个 Program Header 的字节大小 |
| `e_phnum` | ELF Header | Program Header 的数量 |
| `p_type` | Program Header | 段类型 (`PT_LOAD` = 1) |
| `p_flags` | Program Header | 段权限 (`PF_X`=1, `PF_W`=2, `PF_R`=4) |
| `p_offset` | Program Header | 段在文件中的偏移 |
| `p_vaddr` | Program Header | 段的虚拟起始地址 |
| `p_filesz` | Program Header | 段在文件中的大小 |
| `p_memsz` | Program Header | 段在内存中的大小 |
| `p_align` | Program Header | 段对齐要求 (通常 0x1000) |

### 8.2 x86-64 常用机器码速查

| 指令 | 机器码 |
|------|--------|
| `push rax` | `50` |
| `push rdi` | `57` |
| `pop rax` | `58` |
| `pop rdi` | `5f` |
| `nop` | `90` |
| `syscall` | `0f 05` |
| `jmp short` | `eb xx` |
| `jmp rel32` | `e9 xxxx xxxx` |
| `call rel32` | `e8 xxxx xxxx` |
| `mov eax, imm` | `b8 imm32` |
| `mov edi, imm` | `bf imm32` |
| `mov edx, imm` | `ba imm32` |
| `push r8` | `41 50` |
| `push r15` | `41 57` |
| `pop r8` | `41 58` |
| `pop r15` | `41 5f` |

### 8.3 Syscall 速查 (x86-64 Linux)

| 调用号 | 名称 | rdi | rsi | rdx |
|--------|------|-----|-----|-----|
| 0 | read | fd | buf | count |
| 1 | write | fd | buf | count |
| 60 | exit | status | — | — |
| 231 | exit_group | status | — | — |

### 8.4 注入技术对比

| 技术 | 注入位置 | 空间大小 | 隐蔽性 | 修改内容 |
|------|---------|---------|--------|---------|
| Segment Padding | LOAD 段末尾 gap | 数十~数KB | 中 | e_entry, p_filesz, p_memsz |
| 新增 Section | 文件末尾 | 任意大 | 低 | 添加 section header, segment |
| GOT/PLT Hook | GOT/PLT 条目 | 每条目 8B | 高 | 单个函数指针 |
| DT_NEEDED | 动态段 | N/A | 高 | 添加依赖库 |
| init_array | init 函数表 | 每条目 8B | 高 | 函数指针 |

---

> **免责声明**：本文档仅供学习交流 ELF 文件格式和二进制安全研究使用。请勿将相关技术用于任何未经授权的恶意用途。
