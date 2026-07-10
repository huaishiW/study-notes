Bochs 在进入 `cpu_loop()` 真正开始模拟指令之前，需要完成一套极为严密的初始化流程。这个过程可以概括为依次调用的子函数链：**`bxmain()` → `bx_init_main()` → `bx_init_options()` → `bx_init_hardware()` → `bx_reset_cpu()` → `BX_MEM(0)->load_ROM()` → ... → `bx_begin_simulation()` → 最终调用 `cpu_loop()`**。

下面是每个阶段的具体任务。

### 🧩 第一阶段：解析命令行与配置文件

这是程序入口处理的事情，目的是获取用户设置的运行参数。

- **程序入口**：在 `main.cpp` 中，平台相关的入口（如 `main()`、`WinMain()`）最终都调用核心的 `bxmain()` 函数来启动一切[](https://www.cnblogs.com/zeng2013/p/3404379.html)。
    
- **初始化模拟器核心**：`bxmain()` 调用 `bx_init_siminterface()`，创建全局唯一的模拟器接口对象 `SIM`，它是负责配置与通信的"大总管"[](https://blog.csdn.net/weixin_30919571/article/details/96529032)。
    
- **解析命令行**：调用 `bx_init_main()`，处理 `-q`（快速启动）、`-f bochsrc`（指定配置文件）等命令行参数[](https://blog.csdn.net/weixin_30919571/article/details/96529032)。
    
- **解析配置文件**：通过 `SIM` 对象调用 `configuration_interface` 函数，解析 `.bochsrc` 文件中的配置项[](https://blog.csdn.net/weixin_30919571/article/details/96529032)。
    

### 🛠️ 第二阶段：初始化模拟器核心硬件（`bx_init_hardware()`）

这个阶段由 `bx_init_hardware()` 函数主导，是初始化最核心的部分，负责将配置解析得到的信息，转化为内存中的虚拟硬件。

1. **构建配置参数树**：`bxx_init_options()` 负责建立系统的配置参数树，将解析好的配置信息传递给后续的硬件初始化模块[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)。
    
2. **初始化CPU**：`bx_init_cpu()` 对 CPU 模拟核心进行初始化，准备指令译码表等核心结构[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)。
    
3. **初始化内存**：`BX_MEM(0)->init_memory(memSize)` 根据配置向系统申请内存。Bochs 会严格划分出主内存、BIOS ROM 空间等区域[](https://www.cnblogs.com/zeng2013/p/3409859.html)。
    
4. **初始化设备与插件**：初始化 PIC（中断控制器）、PIT（定时器）、键盘、硬盘等所有配置的 I/O 设备。若启用了插件系统，还会调用 `BX_INIT_PLUGINS()` 以支持动态加载设备模块。
    
5. **创建GUI界面**：根据配置选择合适的 GUI 插件。在 Windows 下通过 `IMPLEMENT_GUI_PLUGIN_CODE(win32)` 创建窗口和**独立显示线程**，将模拟主线程与界面线程分离[](https://blog.csdn.net/weixin_30919571/article/details/96529032)。
    

### 🧬 第三阶段：加载BIOS与创建内存

- **加载ROM**：`BX_MEM(0)->load_ROM()` 将模拟的 BIOS 文件（如 `BIOS-bochs-latest`）和 VGA BIOS 文件“烧写”到之前分配好的 ROM 内存区域[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)[](https://www.cnblogs.com/zeng2013/p/3409859.html)。
    
- **创建CPU对象**：调用 `newCPU()` 函数创建 CPU 类的具体实例，准备进行核心模拟任务。
    

### ⚡ 第四阶段：初始化CPU与设备最终状态

- **初始化CPU**：调用 `bx_init_cpu()` 为CPU设置初始状态，准备指令译码表等核心结构[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)。
    
- **硬件复位**：`bx_reset_cpu()` 执行一次硬件复位，将 CPU 的寄存器设置为上电初始值（最关键的是 `EIP` 指针被设为 `0x0000FFF0`），所有 I/O 设备回到初始状态[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)。
    

### 🌐 第五阶段：启动交互界面

- **显示交互界面**：`SIM->configuration_interface(ci_name, CI_START)` 启动在第二阶段创建好的配置界面（文本模式或 GUI 模式），等待用户的启动指令[](https://blog.csdn.net/weixin_30919571/article/details/96529032)[](https://www.cnblogs.com/zeng2013/p/3404379.html)。
    

### 🎬 第六阶段：执行硬件复位

- **硬件复位**：`bx_reset_cpu()` 执行一次硬件复位，将 CPU 的寄存器设置为上电初始值（最关键的是 `EIP` 指针被设为 `0x0000FFF0`），所有 I/O 设备回到初始状态[](https://max.book118.com/html/2019/0320/5214110211002021.shtm)。
    

### 🚀 第七阶段：创建执行线程

- **创建执行线程**：`BX_SCHEDULER(0)->start()` 负责创建真正的模拟执行线程，为进入主循环做最后准备。
    

### ♾️ 第八阶段：进入 `cpu_loop()` 主循环

- **开始模拟**：当用户在界面指令中选择"开始模拟"后，`bx_begin_simulation()` 被调用，最终跳入 `CPU->loop()`，模拟器的心脏开始跳动[](https://blog.csdn.net/weixin_30919571/article/details/96529032)。
    

---

### 💎 总的"执行链路"清单：

为了方便你对照源码，**进入 `cpu_loop()` 之前所经历的具体函数调用顺序**如下：

1. **入口函数**：`main()` 或 `WinMain()` -> **`bxmain()`**
2. 在 `bxmain()` 中：`bx_init_siminterface()` -> **`SIM` 对象创建**
3. 在 `bxmain()` 中：**`bx_init_main()`** (解析命令行)
4. 在 `bxmain()` 中：`SIM->configuration_interface(...)` (解析配置文件)
5. 在配置解析流程中，最终会触发：**`bx_init_hardware()`**
    - 在 `bx_init_hardware()` 中：**`bxx_init_options()`** (构建参数树)
    - 在 `bx_init_hardware()` 中：**`bx_init_cpu()`** (CPU初始化)
    - 在 `bx_init_hardware()` 中：`BX_MEM(0)->init_memory(...)` (内存初始化)
    - 在 `bx_init_hardware()` 中：**`BX_INIT_PLUGINS()`** (设备与插件初始化)
    - 在 `bx_init_hardware()` 中：**`GUI 初始化`** (创建显示线程)
6. **加载ROM**：`BX_MEM(0)->load_ROM(...)`
7. **创建CPU**：`newCPU()`
8. **硬件复位**：`bx_reset_cpu()`
9. **界面启动**：`SIM->configuration_interface(ci_name, CI_START)`
10. **创建线程**：`BX_SCHEDULER(0)->start()`
11. **最终起跳**：`bx_begin_simulation()` -> **`cpu_loop()`**

至此，整个虚拟PC的"骨架"和"器官"都已准备就绪，控制权正式移交给 `cpu_loop()`，开始周而复始的"取指-译码-执行"的模拟心跳。