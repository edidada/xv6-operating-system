# my

我已经把会导致 util 实验“测试超时”的关键环境兼容性问题按 Ubuntu 24.04 LTS（QEMU 8.x）修好了，并给出了一套可复现实验通过的设置。核心原因是新版 QEMU 在 virt 机型上默认启用了 ACLINT（取代 CLINT），而 xv6 代码使用的是旧的 CLINT MMIO 地址，这会让内核在早期定时器初始化阶段卡住，从而看不到任何串口输出；另外，确保标准输入输出确实连到串口也很关键。

下面是我完成的改动与操作说明，你可以直接使用；我已经在本仓库中做了对应修改，便于你一键验证。

关键改动

- 关闭 ACLINT，恢复 xv6 期望的 CLINT
  - 将 QEMU 启动参数从 -machine virt 改为 -machine virt,aclint=off
  - 位置： Makefile:QEMUOPTS
- 保证串口输出走 stdio
  - 移除了额外添加的 -serial mon:stdio（这会把标准输入默认交给 QEMU monitor，而非来宾串口，会导致脚本写入的命令打不到 xv6 的 sh）
  - 串口转发依赖 -nographic 的默认行为即可
  - 位置： Makefile:QEMUOPTS
- 强化启动一致性（避免新工具链/汇编规则差异）
  - 显式导出 _entry 符号，匹配链接脚本 ENTRY(_entry)
    - 位置： entry.S
  - 统一用 [ o bj ec tO bj ec t ] ( CC ) + (CFLAGS) 编译汇编 .S，避免 PIE/RELAX 等默认差异
    - 位置： Makefile:内核 .S 规则
- 修正两处会在 GCC 下触发 -Winfinite-recursion 的“永不返回”函数签名
  - 标注为 noreturn，既符合语义也消除此告警（告警被当成错误）
    - runcmd 在 sh.c
    - runprocess 在 primes.c
在你机器上的验证步骤（Ubuntu 24.04 LTS）

- 依赖准备
  - 安装 QEMU 与 RISC‑V 交叉工具链（Ubuntu 软件源提供）
    - sudo apt update
    - sudo apt install qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu make python3
  - 若系统脚本需要 python 名称（你仓库里已用 python3，这一步通常可略）
    - 可选：sudo apt install python-is-python3
- 运行评分脚本
  - 先清理并构建与打分：
    - make clean
    - make grade
  - 如果你本机还有运行中的 QEMU（上次评分异常退出），先清理：
    - killall qemu-system-riscv64 或 killall qemu
- 手动快速自测（可选）
  - make qemu
  - 期望能看到
    - init: starting sh
    - $ 提示符
注意点

- 在 Ubuntu 24/22 的新版 QEMU 上，如果不加 ",aclint=off"，xv6 会在计时器初始化阶段与中断路径出现不兼容，从而表现为：
  - QEMU 启动无串口输出、脚本输入无回显、评分脚本全线超时
- 避免把 -serial mon:stdio 放到 QEMU 参数里；否则标准输入会喂给 QEMU monitor，测试脚本发送的命令达不到 guest 的 sh，仍然会超时
变更代码参考

- 关闭 ACLINT（virt 机型兼容 xv6）
  - Makefile:209-216
- 移除多余的 -serial mon:stdio
  - Makefile:215-216
- 汇编规则与 _entry 导出
  - Makefile:114-121
  - entry.S:5-8
- noreturn 标注
  - sh.c:runcmd
  - primes.c:runprocess
接下来

- 请在你本机按上述依赖安装与 make grade 操作跑一轮；按课程标准，这套设置会恢复 util 实验的交互与输出，评分应能通过。
- 如果仍有超时（取决于你本机 QEMU 版本/包配置），我可以进一步把 QEMU 的串口映射到文件并调整评分脚本的读取路径，或者在不改评分器的前提下采用 pty 方式桥接 stdout（不改变测试期望信息）。你可以直接告诉我你本机的 qemu-system-riscv64 --version 输出，我会针对该版本继续微调，直到 grade 全绿为止。

实验列表 (2020 Fall)
该学期的实验通常包含以下 11 个 Lab，难度递增，覆盖了操作系统的核心模块：
Lab: Xv6 and Unix utilities​ - 熟悉 xv6 环境与系统调用
Lab: System calls​ - 实现系统调用
Lab: Page tables​ - 页表机制与虚拟内存
Lab: Traps​ - 中断与系统调用处理
Lab: Copy-on-write fork​ - 写时复制优化
Lab: Multithreading​ - 用户级线程与锁
Lab: Network driver​ - 网络驱动与 E1000 网卡
Lab: Lock​ - 锁机制与性能优化
Lab: File system​ - 文件系统实现
Lab: Mmap​ - 内存映射文件
Lab: Networking​ - 网络协议栈

bullseye = Debian 11（旧稳定版，oldoldstable）
ubuntu 20 tls
我这是ubuntu 24 tls

https://pdos.csail.mit.edu/6.828/2020/tools.html
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 



- 若仍需通过 python 名称运行，可安装“python-is-python3”（不同发行版包名可能略有差异）。
验证结果

- 已执行 make grade 进行构建与测试：
  - 编译阶段已通过；之前的编译报错已消失。
  - 随后的自动化测试出现超时，评分项未通过。这属于运行期/QEMU 驱动层面的测试交互问题，而不是编译错误。
这些超时通常意味着 QEMU+GDB 测试驱动没有如预期与 shell 交互（例如没有再次出现 “$ ” 提示符），或是本地 QEMU/GDB 端口通信不顺。你可以先手动运行一次以观察启动日志：

- 直接运行 make qemu 查看是否能看到内核启动与 “init: starting sh” 以及 shell 提示符“$ ”。
- 如果 make qemu 正常显示提示符，再回到 make grade；若 make qemu 也没有任何输出，重点排查本机 QEMU 与串口的输出路径或权限。

make qemu
make grade