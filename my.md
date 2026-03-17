# my
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