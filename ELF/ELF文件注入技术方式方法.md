# ELF 文件注入技术详解

## 一、常见的 ELF 注入方式

### 1. 入口点重定向 (Entry Point Redirect)
**原理**：修改 ELF 文件头中的入口点地址(Entry Point)，使其指向注入的代码位置。

**注入步骤**：
1. 在目标二进制中找到一段可用空间(如代码洞穴或新增 section)
2. 将原始入口点地址保存到注入代码的末尾或固定寄存器
3. 在注入代码执行完毕后，跳转回原始入口点继续执行原程序

**优点**：实现简单直接
**缺点**：容易被入口点完整性检测发现

---

### 2. 代码洞穴/填充区注入 (Code Cave / Padding Injection)
**原理**：利用 ELF 可执行段末尾的零填充区域(通常是 .text 段末尾对齐后的空隙)。

**特点**：
- 在 x86_64 架构下，每个 LOAD 段通常按页(4096字节)对齐
- segment 的 `p_filesz` < `p_memsz` 时，末尾会有填充区域
- 可以在这些填充区域中注入 shellcode

**工具参考**：
- `woody_woodpacker`：利用 segment padding 注入 + 自加密自解密
- `elf_injector`：在 text segment padding 中注入任意大小的可重定位代码

---

### 3. 新增 Section 注入 (New Section Injection)
**原理**：在 ELF 文件中创建一个新的 section，用于存放注入的代码。

**注入步骤**：
1. 在 ELF 末尾添加新的 section header
2. 将 shellcode 写入到文件末尾的空闲空间
3. 修改 segment 映射，将新 section 纳入可执行段
4. 修改入口点或利用 constructor/函数指针触发执行

**优点**：注入空间充足，可执行更多功能
**缺点**：需要修改 section header table，容易被检测

---

### 4. GOT/PLT Hook
**原理**：修改 Global Offset Table (GOT) 或 Procedure Linkage Table (PLT) 中的条目。

**GOT Hook**：
- GOT 存储外部函数(如 libc)的真实地址
- 覆盖 GOT 条目，使其指向注入代码
- 下次调用该函数时，执行注入的代码

**PLT Hook**：
- PLT 是外部函数调用的跳板
- 修改 PLT 条目，劫持函数调用

**优点**：针对性强，只影响特定函数调用
**缺点**：需要了解目标二进制使用的外部函数

---

### 5. DT_NEEDED 覆盖注入
**原理**：在动态段(Dynamic Section)中覆盖或添加 `DT_NEEDED` 条目，添加一个恶意的共享库依赖。

**优点**：利用正常的动态链接机制，比较隐蔽
**缺点**：需要宿主库在目标系统存在

---

### 6. .init/.fini 函数注入
**原理**：利用 ELF 的初始化/终止函数机制。

**方式**：
- 修改 `DT_INIT` 或 `DT_FINI` 条目
- 修改 `.init_array` / `.fini_array` 中的函数指针
- 注入代码会在程序 main() 之前(init)或之后(fini)执行

---

### 7. 构造函数/析构函数注入
**原理**：利用 C++ 全局对象的构造函数机制，或 GCC 的 `__attribute__((constructor))`。

**特点**：
- 注入的代码会在 main() 之前自动执行
- 无需修改入口点，更为隐蔽

---

## 二、注入位置通常在哪里

### 常见的注入目标区域

| 位置 | 说明 | 可用空间 |
|------|------|----------|
| **.text 段末尾填充** | 程序代码段末尾的对齐填充区 | 通常几十字节到几KB |
| **segment padding** | LOAD segment 末尾的文件大小与内存大小的差值 | 可达数KB |
| **新增 section** | 文件末尾空闲区域 | 可根据需要扩展 |
| **.data/.bss 空闲区** | 未使用的全局变量空间 | 一般较小 |
| **GOT/PLT 条目** | 外部函数跳转表 | 每条目 8 字节 |
| **.fini_array** | 终止函数数组 | 每函数指针 8 字节(x64) |

### 注入区域的内存属性要求

对于注入代码必须满足：
- **可执行权限** (`PF_X`)：代码需要被 CPU 执行
- **可读权限** (`PF_R`)：代码需要被读取
- 通常需要与原程序处于同一权限域(如同为 user mode)

---

## 三、注入代码的执行触发方式

### 1. 入口点跳转
直接修改 ELF Header 中的 `e_entry`，指向注入代码

### 2. 函数 Hook
- PLT/GOT Hook
- 篡改函数指针

### 3. 初始化函数
- `DT_INIT`
- `.init_array`
- `__attribute__((constructor))`

### 4. 终止函数
- `DT_FINI`
- `.fini_array`
- `__attribute__((destructor))`

### 5. 中断/信号处理
修改信号处理函数指针

### 6. 构造假的符号
添加一个导出函数，被正常调用时触发

---

## 四、elf_injector 详细工作原理

### 4.1 项目概述

**elf_injector** (dillstead/elf_injector) 是一个开源的 ELF 注入工具，其核心特点是利用 **text segment padding** 注入技术，将任意大小的可重定位代码注入到 ELF 可执行文件中。

**项目地址**: https://github.com/dillstead/elf_injector

**核心特点**:
- 注入可重定位(relocatable)的代码块
- 代码在原始入口点(OEP = Original Entry Point)之前执行
- 无需修改目标 ELF 的 section headers

---

### 4.2 注入方式：Segment Padding Injection (段填充注入)

#### 原理

ELF 可执行文件在加载时，操作系统按 **segment (LOAD segment)** 来映射内存。每个 segment 需要按页面大小(通常 4096 字节)对齐。

关键观察：
```
Segment VirtAddr:     0x401000  (.text segment起始)
Segment MemorySize:   0x1763    (实际代码大小)
Segment FileSize:     0x1763
页面对齐边界:         0x401000 + 0x2000 = 0x403000  (假设按2页对齐)
                                              ↑
                         之间的空隙就是 PADDING / CODE CAVE
```

在 x86_64 Linux 中，编译器/链接器通常会在 segment 末尾保留一段 **对齐填充区(unused padding)**，这片区域：
- 在文件中存在但无实际代码/数据
- 具有 **可执行(RX)权限**
- 可被利用来存放注入的 shellcode

#### 注入位置计算

```c
// 从 elf_injector 源码中提取的核心算法
void extractSegmentsBoundaries(Elf64_Ehdr *elf_headers, Elf64_Phdr *prog_headers,
                               Elf64_Xword *newEP, Elf64_Xword *text_segment_end,
                               Elf64_Xword *gap)
{
    for (int i = 0; i < elf_headers->e_phnum; ++i)
    {
        prog_headers = (Elf64_Phdr *)((unsigned char *)prog_headers +
                                       elf_headers->e_phentsize);

        if (prog_headers->p_type == PT_LOAD)
        {
            // 标记为可写 (原本可能只是可读可执行)
            prog_headers->p_flags |= PF_W;

            // 找到同时具有 RWX 权限的 segment (.text segment)
            if (prog_headers->p_flags == (PF_W | PF_X | PF_R))
            {
                // 新的入口点 = segment虚拟地址 + segment文件大小
                // 这里就是 code cave 的起始位置
                *newEP = prog_headers->p_vaddr + prog_headers->p_filesz;

                // 记录 segment 在文件中的偏移
                *text_segment_end = prog_headers->p_filesz;

                // gap = segment对齐边界末尾 - segment实际数据末尾
                // 这里的 gap 就是可注入代码的最大空间
            }
        }
    }
}
```

**核心公式**:
```
code_cave_start = segment_vaddr + segment_file_size
available_space = 页对齐边界 - (segment_vaddr + segment_file_size)
```

---

### 4.3 工作流程详解

#### 第一步：解析 ELF 头部和程序头表

```c
// 读取目标 ELF 文件到内存
char *content = mapFile(path, &filesize);

// 解析 ELF Header
Elf64_Ehdr *elf_headers = (Elf64_Ehdr *)content;

// 解析 Program Header Table (段头表)
Elf64_Phdr *prog_headers = (Elf64_Phdr *)(content + elf_headers->e_phoff);

// 解析 Section Header Table (节头表) - 用于找到 .text section
Elf64_Shdr *section_headers = (Elf64_Shdr *)(content + elf_headers->e_shoff);
```

#### 第二步：提取段边界信息

遍历所有 program headers，找到 **PT_LOAD** 类型的段，并确定：

| 值 | 含义 |
|----|------|
| `e_entry` | 原始入口点 (OEP) |
| `p_vaddr + p_filesz` | text segment 末尾 = 新入口点候选位置 |
| `p_offset + p_filesz` | text segment 在文件中的结束偏移 |
| gap 空间 | 可注入代码的最大字节数 |

#### 第三步：检查可用空间

```c
// shellcode 必须能放入 code cave 中
if (shellcode_size > gap) {
    perror("[-] The shellcode size is larger than gap");
    exit(EXIT_FAILURE);
}
```

#### 第四步：定位 .text section 信息

```c
// 从 section headers 中找到 .text section
parseSections(elf_headers, section_headers, section_mem,
              &text_sec_init, &text_sec_end, &text_sec_size, ".text");

// text_sec_init = .text section 的起始虚拟地址
// text_sec_end  = .text section 的结束虚拟地址
```

#### 第五步：修改权限并扩展 segment

```c
// 给 text segment 添加写权限 (原本只有 RX)
prog_headers->p_flags |= PF_W;

// 关键：扩展 text segment 的 p_filesz 和 p_memsz
// 这样注入的代码才会被纳入可执行内存区域
extendTextSegment(elf_headers, prog_headers, shell_text_sec_size);
```

**extendTextSegment 的作用**:
```c
void extendTextSegment(Elf64_Ehdr *elf_headers, Elf64_Phdr *prog_headers,
                       Elf64_Xword additional_size)
{
    // 1. 扩展 segment 的文件大小和内存大小
    prog_headers->p_filesz += additional_size;
    prog_headers->p_memsz  += additional_size;

    // 2. 修改 e_entry 指向注入代码位置
    elf_headers->e_entry = prog_headers->p_vaddr + prog_headers->p_filesz - additional_size;
}
```

#### 第六步：写入 shellcode

将 shellcode 的机器码写入 text segment 末尾的 code cave 位置：

```c
// content + text_segment_end = code cave 在文件中的位置
memcpy(content + text_segment_end, shellcodeContent, shellcode_size);
```

#### 第七步：保存原始入口点

注入的代码执行完毕后，需要跳转回原始程序入口点继续执行。跳转指令(jmp)通常被嵌入在 shellcode 的末尾，或者原始入口点地址被传递给注入代码。

**注入代码的结构**:
```
+------------------+
|  注入的 shellcode |   <- 新入口点 (e_entry 指向这里)
+------------------+
|  jmp original_ep |   <- 跳转到原始入口点
+------------------+
```

---

### 4.4 完整算法流程图

```
┌──────────────────────────────────────────────────────────┐
│                   elf_injector 工作流程                    │
└──────────────────────────────────────────────────────────┘

1. 读取目标 ELF 文件到内存
           │
           ▼
2. 解析 ELF Header、Program Headers、Section Headers
           │
           ▼
3. 遍历 Program Headers，找到 PT_LOAD 类型的 segment
           │
           ▼
4. 判断该 segment 是否具有 RWX 权限 (text segment)
           │
           ├─ 是 ──► 5a. 计算 code cave 起始地址和可用空间
           │              new_ep = p_vaddr + p_filesz
           │              gap = 页对齐边界 - new_ep
           │
           └─ 否 ──► 继续遍历下一个 segment
           │
           ▼
6. 验证 shellcode 大小 <= gap
           │
           ▼
7. 修改 segment 权限: 添加 PF_W (写权限)
           │
           ▼
8. 扩展 text segment:
           │   - p_filesz += shellcode_size
           │   - p_memsz  += shellcode_size
           │
           ▼
9. 修改 ELF Header:
           │   - e_entry = 指向注入代码位置
           │
           ▼
10. 将 shellcode 写入 text segment 末尾的 code cave
           │
           ▼
11. 保存修改后的 ELF 文件
```

---

### 4.5 注入前后对比

**注入前 (ELF Header)**:
```
Entry point address: 0x401000
Program Headers:
  Type   Offset   VirtAddr   PhysAddr   FileSiz  MemSiz   Flg  Align
  LOAD   0x000000 0x00400000 0x00400000 0x15000  0x15000  R E  0x1000
                         ↑
                    e_entry = 0x401000
```

**注入后 (ELF Header)**:
```
Entry point address: 0x401500   ← 修改为 code cave 地址
Program Headers:
  Type   Offset   VirtAddr   PhysAddr   FileSiz  MemSiz   Flg  Align
  LOAD   0x000000 0x00400000 0x00400000 0x15500  0x15500  RWX  0x1000
                                                      ↑
                                              文件大小扩展了
                         ↑         ↑
                         │         └─── 原有代码 + code cave + shellcode
                         └─── 新入口点指向这里
```

---

### 4.6 注入代码的执行流程

```
程序启动
    │
    ▼
CPU 从 e_entry 指定的地址开始执行
    │
    ▼
执行注入的 shellcode (在 code cave 中)
    │
    ├──> 执行恶意功能 (如: 打开后门、读取敏感数据等)
    │
    ▼
执行跳转指令 (jmp) 回到原始入口点
    │
    ▼
继续执行原始程序的正常代码
    │
    ▼
程序正常退出
```

---

### 4.7 关键技术细节

#### (1) 为什么需要 p_flags |= PF_W ?

text segment 原本通常是 **RX** (只读可执行)，不允许写入。通过添加 **PF_W** 标志，使操作系统允许写入这片内存区域(用于存放注入代码)。

#### (2) 为什么扩展 p_filesz 和 p_memsz ?

如果不扩展这两个字段：
- 操作系统只会加载原始大小的内容到内存
- 注入的代码虽然写入了文件，但不会被加载到内存，CPU 无法执行

#### (3) 为什么不会破坏原有程序?

- shellcode 被写入到 text segment **末尾** 的 code cave
- 原有代码区域完全不受影响
- shellcode 执行完毕后跳回 OEP，程序逻辑完整

#### (4) 局限性与检测

- **空间有限**: 取决于 page aligned 后产生的 padding 大小
- **入口点变化**: `e_entry` 不再指向 `.text` section 起始，容易被检测
- **权限变化**: text segment 从 RX 变成 RWX，违反最小权限原则

---

### 4.8 其他类似项目对比

| 项目 | 注入技术 | 特点 |
|------|----------|------|
| **elf_injector** | Segment padding | 注入可重定位代码，修改入口点 |
| **woody_woodpacker** | Segment padding | 注入 + 自加密 |
| **elfspirit** | 多种技术 | 静态分析 + 注入框架 |
| **E9Patch** | 插入新 segment | 强大的静态二进制重写 |

---

## 五、工具推荐

| 工具 | 类型 | 特点 |
|------|------|------|
| **elf_injector** | 开源 | 注入可重定位代码块，在原始入口点之前执行 |
| **woody_woodpacker** | 开源 | segment padding 注入 + 加密 |
| **E9Patch** | 开源 | 强大的静态二进制重写工具 |
| **LIEF** | 库 | C++/Python 库，用于 ELF 二进制操作 |
| **rootkit (LKM)** | 内核模块 | 运行时动态注入 |

---

## 六、检测与防护(参考)

- **入口点完整性校验**：检测 e_entry 是否被篡改
- **Section 差异对比**：对比干净版本与运行版本的 section
- **GOT/PLT 完整性监控**：监控 GOT/PLT 条目修改
- **内存完整性**(SELinux/AppArmor)：限制内存区域的写权限
- **签名校验**：对二进制文件进行签名，运行时校验

---

## 七、技术要点总结

```
ELF 注入的核心步骤:
┌─────────────────────────────────────────────────────────┐
│  1. 分析目标 ELF 结构 (readelf, objdump, r2)           │
│  2. 确定注入点 (text padding / new section / GOT)    │
│  3. 编写/准备注入代码 (shellcode)                      │
│  4. 修改 ELF 二进制 (写入代码 + 修改头部/表)           │
│  5. 设置触发机制 (入口点/Hook/init function)           │
│  6. 测试验证 (执行并恢复原流程)                        │
└─────────────────────────────────────────────────────────┘
```

---

> **免责声明**：本文档仅供学习交流 ELF 文件格式和二进制安全研究使用。请勿将相关技术用于任何未经授权的恶意用途。
