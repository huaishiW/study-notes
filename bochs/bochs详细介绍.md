# Bochs 模拟器详解

## 什么是 Bochs

Bochs（发音为 "box"）是一款便携式 IA-32 和 x86-64 IBM PC 兼容模拟器（Emulator）和调试器（Debugger），主要使用 C++ 编写，并根据 GNU Lesser General Public License（LGPL）开源发布。

与 VMware 或 VirtualBox 等虚拟机不同，Bochs 通过软件模拟完整硬件，这意味着它能够模拟从处理器到外设的每一个硬件组件。由于这种高度逼真的模拟，Bochs 常用于操作系统开发、驱动调试以及运行那些无法在现代硬件上运行的老旧软件。

---

## 历史背景

| 时间 | 事件 |
|------|------|
| **1994 年** | Bochs 由 Kevin Lawton 创立，最初以 25 美元的商业许可销售 |
| **2000 年 3 月 22 日** | Mandrakesoft（后更名为 Mandriva）收购 Bochs，并将其以 LGPL 协议开源 |
| **2025 年 2 月 16 日** | 发布稳定版本 3.0 |

Bochs 的开源转型使其成为社区驱动的项目，吸引了全球开发者参与维护和贡献。

---

## 技术架构

### 跨平台支持

Bochs 具有出色的跨平台特性，支持多种操作系统作为宿主机（Host OS）：

- Linux
- BSD（FreeBSD、OpenBSD、NetBSD）
- macOS
- Windows
- Windows CE
- OS/2
- BeOS
- MorphOS
- AmigaOS
- Android
- PlayStation 2

### CPU 模拟

- 可模拟多达 **8 个 CPU**（对称多处理，SMP）
- 支持 IA-32（i386、i486、Pentium 系列）
- 支持 x86-64（AMD64/Intel 64）架构
- 模拟 8086、80286、80386、80486、Pentium、Pentium Pro 等处理器

### 可模拟硬件组件

| 类别 | 支持的硬件 |
|------|-----------|
| **显卡** | Cirrus Logic CL-GD5430/5446、3dfx Voodoo Banshee/Voodoo3 |
| **声卡** | Sound Blaster 16、ES1370 |
| **网络** | NE2000、Intel 82540EM 千兆以太网适配器 |
| **芯片组** | Intel 430FX/440FX/440BX、PIIX3/PIIX4 |
| **存储** | 软盘驱动器（Floppy）、CD-ROM 驱动器、硬盘 |
| **USB** | USB 1.1/2.0 支持 |
| **其他** | 键盘、鼠标、串口、并口、PCI 总线 |

---

## 主要用途

### 1. 操作系统开发与调试

Bochs 最主要的应用场景之一是操作系统开发。由于模拟的操作系统崩溃不会影响宿主机，开发者可以在安全的环境中：

- 调试内核代码
- 开发设备驱动程序
- 单步跟踪操作系统启动过程
- 分析系统级问题

### 2. 运行旧软件和游戏

许多在现代计算机上无法运行的老旧软件和游戏，可以通过 Bochs 模拟的旧硬件环境完美运行：

- DOS 游戏和应用程序
- 早期 Windows 版本（如 Windows 95/98/ME）
- 只能在特定硬件配置下运行的软件

### 3. 多系统测试

开发者可以使用 Bochs 在已运行的宿主机上测试各种客户操作系统（Guest OS），无需额外的物理硬件。

### 4. 教学与研究

Bochs 详细揭示了 PC 硬件的工作原理，是计算机体系结构教学的理想工具。

---

## 支持的操作系统

Bochs 支持以下客户操作系统（Guest OS）：

| 操作系统类别 | 具体版本 |
|-------------|----------|
| **DOS** | DOS 6.22 及更早版本 |
| **Linux** | 多种发行版（Slackware、Debian、Red Hat 等） |
| **Windows** | Windows 3.1、Windows 95、Windows 98、Windows ME、Windows NT 4.0、Windows 2000、Windows XP |
| **BSD** | FreeBSD、OpenBSD、NetBSD |
| **其他** | Xenix、386BSD、Rhapsody OS（Mac OS X Public Beta 的前身） |

---

## Bochs 与其他虚拟化技术的区别

| 特性        | Bochs       | VMware / VirtualBox | QEMU（KVM 模式） |
| --------- | ----------- | ------------------- | ------------ |
| **模拟方式**  | 软件模拟（完全虚拟化） | 半虚拟化 / 全虚拟化         | 混合模式         |
| **性能**    | 较慢          | 较快                  | 较快           |
| **硬件逼真度** | 极高          | 较高                  | 中等           |
| **调试能力**  | 强大的内置调试器    | 有限                  | 中等           |
| **资源占用**  | 较高          | 中等                  | 较低           |

Bochs 的软件模拟方式虽然性能较低，但其**极高的硬件逼真度**和**强大的调试能力**使其在特定场景下不可替代。

---

## 主要特性总结

- **开源免费**：LGPL 协议，完全免费使用
- **高度可移植**：支持 20+ 操作系统平台
- **完整硬件模拟**：从 CPU 到声卡、显卡、网络卡等
- **内置调试器**：支持单步调试、断点设置、内存查看等
- **多 CPU 支持**：最多模拟 8 个处理器
- **广泛操作系统支持**：支持从 DOS 到 Windows XP 的多种操作系统

---

## 参考链接

- [Bochs 官方网站](https://bochs.sourceforge.io/)
- [Wikipedia - Bochs](https://en.wikipedia.org/wiki/Bochs)
- [SourceForge 项目页面](https://sourceforge.net/projects/bochs/)

---

*本文档由 AI 生成，整理自公开资料。最后更新：2026 年 4 月。*
