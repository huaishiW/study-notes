``` C
#include <cstdio>
#include "wucpu.h"
#include "memory-bochs.h"
#include "logio.h"
#include "pc_system.h"
#include "iodev.h"
#include "cmos.h"
#include <signal.h>

bool bx_user_quit;
bx_pc_system_c bx_pc_system;

#if BX_SUPPORT_APIC
Bit32u apic_id_mask; // determinted by XAPIC option
#endif

bool simulate_xapic;
BOCHSAPI BX_CPU_C bx_cpu;
BOCHSAPI BX_MEM_C bx_mem;

#if BX_SHOW_IPS
void bx_show_ips_handler(void)
{
    static Bit64u ticks_count = 0;
    static Bit64u counts = 0;
    // amount of system ticks passed from last time the handler was called
    Bit64u ips_count = bx_pc_system.time_ticks() - ticks_count;
    if (ips_count)
    {
        bx_gui->show_ips((Bit32u)ips_count);
        ticks_count = bx_pc_system.time_ticks();
        counts++;
        if (bx_dbg.print_timestamps)
        {
            printf("IPS: %u\taverage = %u\t\t(%us)\n",(unsigned)ips_count, (unsigned)(ticks_count / counts), (unsigned)counts);
            fflush(stdout);
        }
    }
    return;
}
#endif

void CDECL bx_signal_handler(int signum)
{
    // in a multithreaded environment, a signal such as SIGINT can be sent to all
    // threads.  This function is only intended to handle signals in the
    // simulator thread.  It will simply return if called from any other thread.
    // Otherwise the BX_PANIC() below can be called in multiple threads at
    // once, leading to multiple threads trying to display a dialog box,
    // leading to GUI deadlock.
    if (!SIM->is_sim_thread())
    {
        BX_INFO(("bx_signal_handler: ignored sig %d because it wasn't called from the simulator thread", signum));
        return;
    }
#if BX_GUI_SIGHANDLER
    if (bx_gui_sighandler)
    {
        // GUI signal handler gets first priority, if the mask says it's wanted
        if ((1 << signum) & bx_gui->get_sighandler_mask())
        {
            bx_gui->sighandler(signum);
            return;
        }
    }
#endif

#if BX_SHOW_IPS
    if (signum == SIGALRM)
    {
        bx_show_ips_handler();
#if !defined(WIN32)
        if (!SIM->is_wx_selected())
        {
            signal(SIGALRM, bx_signal_handler);
            alarm(1);
        }
#endif
        return;
    }
#endif

#if BX_GUI_SIGHANDLER
    if (bx_gui_sighandler)
    {
        if ((1 << signum) & bx_gui->get_sighandler_mask())
        {
            bx_gui->sighandler(signum);
            return;
        }
    }
#endif
    BX_PANIC(("SIGNAL %u caught", signum));
}

void init(void)
{
    plugin_startup();
    bx_pc_system.initialize(15000000);
    BX_MEM(0)->init_memory(0x0000000002000000, 0x0000000002000000, 0x00020000);
    BX_MEM(0)->load_ROM("BIOS-bochs-latest", 0, 0);
#if BX_SUPPORT_SMP == 0
    BX_CPU(0)->initialize();
    BX_CPU(0)->sanity_checks();
    BX_CPU(0)->register_state();
    BX_INSTR_INITIALIZE(0);
#endif
    DEV_init_devices();
    bx_pc_system.Reset(BX_RESET_HARDWARE);
    bx_gui->init_signal_handlers();
    bx_pc_system.start_timers();
    signal(SIGINT, bx_signal_handler);
}

int main(void)
{
    printf("the func:%s is called......\n", __func__);
    // 初始化
    init();
    // 循环
    BX_CPU(0)->cpu_loop();
    return 0;
}
// 运行程序: Ctrl + F5 或调试 >“开始执行(不调试)”菜单
// 调试程序: F5 或调试 >“开始调试”菜单
// 入门使用技巧:
//   1. 使用解决方案资源管理器窗口添加/管理文件
//   2. 使用团队资源管理器窗口连接到源代码管理
//   3. 使用输出窗口查看生成输出和其他消息
//   4. 使用错误列表窗口查看错误
//   5. 转到“项目”>“添加新项”以创建新的代码文件，或转到“项目”>“添加现有项”以将现有代码文件添加到项目
//   6. 将来，若要再次打开此项目，请转到“文件”>“打开”>“项目”并选择 .sln 文件
```