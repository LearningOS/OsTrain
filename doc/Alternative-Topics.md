# 2020年秋季学期操作系统专题训练课的备选实验题目

当有同学选择某个题目后，老师和助教会进一步完善题目描述有参考文献。

### 已有参考

软件所：[https://github.com/rcore-os/rCore/wiki/OSPP-2020](https://github.com/rcore-os/rCore/wiki/OSPP-2020)

鹏城实习：[https://github.com/rcore-os/zCore/wiki/zcore-summer-of-code](https://github.com/rcore-os/zCore/wiki/zcore-summer-of-code)

去年专题训练：[http://os.cs.tsinghua.edu.cn/oscourse/OsTrain2019](http://os.cs.tsinghua.edu.cn/oscourse/OsTrain2019)

### **CPU测试**

测试程序的执行时间时，程序的执行时间会出现两个稳定的时间点；

关中断的CPU上的执行时间会稳定在两个点上；可能的原因，CPU的流水导致差异；

结果：这个时间的确定性是不成立的；

与OS的关系：屏蔽干扰的影响；

#### 分析

没有if语句就没有这种现象了。

源代码：

```c++
const int P = 998244353;
void add(int &a, int b) {
  a += b;
  if (a >= P) {
    a -= P;
  }
}
int main() {
  int ans = 0;
  for (int i = 0; i < 1000000; i++) {
    add(ans, i);
  }
}
```
对应的汇编代码

![图片](https://uploader.shimo.im/f/g4HpEF1eoY76xvOc.jpg!thumbnail)

出现了85.06ms和92.96ms两个时间；

进一步测试的结果：与8字节和16字节对齐有关系；

确定只可以出现的最小程序：

```c++
__asm__ (
" .text\n"
" .globl main\n"
"main:\n"
" xorl %eax, %eax\n"
" xorl %esi, %esi\n"
" jmp .L6\n" // 这条指令存在才能复现
".p2align 4\n" // >=4: 能复现
".L8:\n"
" addl %eax, %esi\n" // sum += i;
" leal -998244353(%rsi), %edx\n" // temp = sum - MOD;
" cmpl $998244352, %esi\n" // flag = sum >= MOD;
" cmovg %edx, %esi\n" // if (flag) sum = temp;
".L6:\n"
" addl $1, %eax\n" // i++;
" cmpl $100000000, %eax\n" // flag = i >= 1e8;
" jne .L8\n" // if (!flag) goto L8;
" xorl %eax, %eax\n"
" ret\n"
);
```
### rCore/ zCore改进

主题：async 与 IO

#### 恢复 rCore 的网络功能，运行 nginx

rCore 在被杰哥用 async 重构过后，遗留了网络部分功能还不可用。

我们需要用 async 重新适配网卡驱动、网络协议栈、socket 系统调用。

复现杰哥当年 Nginx 性能测试并与 Linux 对比的工作，并尝试进行优化。

#### rCore 文件系统和块设备层的 async 改造

目前磁盘数据的读取是阻塞轮询实现的，其上的通用块层也很简陋，INode 接口也没有彻底支持 async。

#### 在 rCore 中实现 io_uring

io_uring 是 Linux 5.1 新引入的高性能异步 IO 机制。通过学习和实现这一机制，我们可以深入理解最前沿的异步系统调用接口的设计。

主题：图形与 GUI

#### rCore 中 MiniGUI 的支持完善

目前只能显示界面（扫雷），需要继续支持好鼠标和键盘。

刘丰源可以支持

#### 树莓派显卡驱动的继续完善

继承OSTrain2019的工作，也许可以适配树莓派4上的新显卡

主题：移植

#### RISC-V上的zCore移植

车春池可以继续做；

优先把Linux搞定，再有兴趣，就可以做Fushsia（能学到东西）；

#### seL4上的zCore移植

用seL4的好处是，与硬件的相关性减弱；

#### ARM64和树莓派4上的zCore移植

贾越凯：有兴趣，可以提供一些帮助；

ARM有4个特权；谷歌有ARM有官方支持；

学术界支持RV；工业界目前看好ARM；

在ARM上用UEFI，然后移植zCore；

#### x86裸机上的多核支持与优化

目前已经可以 boot 启动所有核，但是无法正确多核运行用户程序，因为没有实现 TLB shutdown。

接下来，可以考虑参考用户态 async 运行时 tokio / async_std / smol，将它们改造为适合内核多核环境的运行时，以优化性能。


主题：教学文档

#### zCore教学实验设计和文档撰写

王润基：先整理一下已有的文档；

#### K210上的rCore多核支持完善和实验文档改进

#### 用rust重写操作系统的各功能模块

### 高可靠支持

#### 内核模块可动态更新的zcore

实现任意内核模块可动态更新与重构

论文：


* OSDI2020论文：Theseus: an experiment in operating system structure and state management
* [Theseus: A State Spill-free Operating System, PLOS'17](http://www.owlnet.rice.edu/~kevinaboos/docs/theseus_plos2017.pdf)
* [A Characterization of State Spill in Modern Operating Systems, EuroSys 2017](http://www.owlnet.rice.edu/~kevinaboos/docs/statespy_eurosys2017.pdf)

代码&文档：


* [https://github.com/theseus-os/Theseus](https://github.com/theseus-os/Theseus)
### **异步支持**

#### **参考资料**

[https://rustcc.cn/article?id=2a02d42f-4b27-40f1-ad0e-2015d3413bb7](https://rustcc.cn/article?id=2a02d42f-4b27-40f1-ad0e-2015d3413bb7)

smol - 异步rumtime

[https://rustcc.cn/article?id=0117ce5f-2c89-49bf-8b06-82bf66acf936](https://rustcc.cn/article?id=0117ce5f-2c89-49bf-8b06-82bf66acf936)

Rust 异步入门

[https://teh-cmc.github.io/rust-async/html/](https://teh-cmc.github.io/rust-async/html/)

系统调用的异步：

线程调度

协程调度

#### 可能的测试小用例

多文件的异步读写；

网络的异步访问；

#### 解释

事件队列：网络收发包事件、磁盘I/O事件、时钟事件

处理事件的协程：用协程按优先级处理事件队列中的待处理事件；（用一个核来处理协程）

中断：用多线程进行处理，其优先级高于协程的执行；（用一个核来专门处理中断）

系统调用接口：在同步场景下，系统调用会阻塞（原因是，它们是一个线程）；在异步系统调用场景下，这个阻塞就没有了，但需要两个单向（应用到内核、内核到应用）的通知机制；

#### *异步系统调用接口

RTC

应用可以提交请求；

应用可以查询请求的响应情况；

内核可以接收应用的请求；

内核可以通知应用的请求响应结果；

可能的做法：共享队列；中断机制；（减少系统调用的次数、减少系统调用的开销）

可能思路：RustSBI接口的异步改进，用串口验证异步的高性能执行；

#### 进程、线程和协程的切换开销小

进程：页表切换开销；

线程：寄存器保存和恢复；栈的切换；

协程：可以理解为函数调用；协程没有栈切换，但有栈帧的内容清掉（这里是编译器处理的），把需要保存的信息放到堆中；

近期目标：在rCore实现内核线程和异步状态机的友好；可能的测例：

#### 同步和异步系统调用的同时存在问题

先有同步系统调用，然后才有部分系统调用支持；

#### vDSO的使用

#### 后续工作


1. 王润基、王逸松：一起写异步的方案文档；
### **内核调度**

### **NVM**

NVM的并行和性能测试，进而对文件系统和I/O的优化；

用NVM来处理崩溃一致性和恢复；

NVM的容量：目前市面上共有两代产品（2020Q2上市的200系列，需要三代至强处理器支持；2019Q2上市的傲腾持久内存，需要二代至强处理器支持），每代产品有三种规格，512GB、256GB和128GB；存储密度高于DRAM（主流容量为32GB/64GB）

NVM的价格：512GB ~6万元人民币；256GB ~2万元人民币；128GB ~4000元人民币

NVM的性能：表中二代即200系列，有第三方测试表明，傲腾的延迟是DDR4的4倍；Intel自己披露的数据，应用一代产品，SAP HANA的重新启动时间（3TB DRAM + 6TB傲腾）从只使用DRAM时的20分钟提高到90秒；Windows Server 2019/Hyper-V的虚拟机支撑能力从只用DRAM（768GB DDR4）时的22个提升至30个（192GB DDR4 + 1TB傲腾）。使用二代产品，Oracle数据库的存内查询性能最高可提升10倍；VMWare Horizon on vSAN可多服务87%的VDI（Virtual desktop infrastructure）用户；相比DRAM+HDD方案，Spark的性能提升最高可达8倍。

|    |读/写带宽（15W 256B，GB/s）|    |    |读/写带宽（15W 64B）|    |    |
|:----|:----:|:----|:----:|:----|:----:|:----|:----:|:----|:----:|:----|:----:|:----|
|    |128GB|256GB|512GB|128GB|256GB|512GB|
|一代|6.8/1.85|6.8/2.3|5.3/1.89|1.7/0.45|1.75/0.58|1.4/0.47|
|二代|7.45/2.25|8.10/3.15|7.45/2.60|1.86/0.56|2.03/0.79|1.86/0.65|

NVM的寿命(EDURANCE)：有限质保5年，一般用质保期内能够写入的数据总量(常用单位是TBW/PBW，即Terabytes/Petabytes Written)或者每天允许的整设备写入次数DWPD(Device Writes Per Day)表示。一代傲腾128GB/256GB/512GB产品在100% write 15W 256B粒度条件下，对应的寿命分别为292/363/300PBW， 64B粒度则分别为91/91/75PBW；二代产品256B粒度条件下为292/497/410PBW，64B粒度则为73/125/103PBW，读写混合条件下，寿命会降低，67%读+33%写情况下，寿命最高降低~41%。

NVM的组织方式：与DRAM使用同样的插槽，与DRAM必须按照1：1进行交叉配比，即DRAM和傲腾必须按照DRAM/NVM/DRAM/NVM...的形式插到主板的插槽中。

#### NVM的性能测试

傲腾持久内存有2种操作模式，即内存模式（Memory Mode）和应用模式（App Direct Mode）。在内存模式中，持久内存直接作为DRAM的扩展使用（操作系统/CPU内存控制器将其视为DRAM，忽略其持久性，而DRAM则被用作Cache），应用无需修改，适合于虚拟化以及I/O受限的应用。在应用模式中，操作系统将NVM和DRAM视作不同的存储层次，NVM的地位等同于现在的硬盘，在具体的编程模型（使用方式）上，又分成两种情况：一是采用特定的编程接口，主要是PMDK（Persistent Memory Development Kit）【注：Intel自身开发的SPDK（Storage Performance Development Kit ）是否支持最新的傲腾持久内存尚需进一步的确认】，直接实现对NVM的访问，从而充分利用NVM的特性，但这意味着要对软件和应用进行修改；二是采用标准的文件APIs访问NVM（称之为Storage over App Direct），无需对应用和文件系统进行修改，可以像SSD那样直接启动OS，但由于避免了数据在I/O总线上的传输，因而具有更高的性能。

任务内容：

1. 测试两种操作模式下不同编程模型或使用方式所能达到的性能，包括但不限于读写带宽、延迟、IOPS等；

2. 从软件栈的角度分析不同操作模式和使用方式性能差异的原因，找出潜在的性能瓶颈。

#### NVM的优化

1.对于前面性能测试过程中找出的瓶颈给出可能的改进方案；

2. 编码实现自己提出的改进方案，并做进一步的性能测试。

#### NVM的崩溃一致性

老问题，一直存在，如数据的部分写——需要原子操作的数据写流程因故障只完成一部分操作；对NVM而言，其持久性遇上乱序写使得问题变得更加严重：乱序写是当今系统的固有属性，对DRAM而言，因掉电或故障重启后数据丢失，不存在问题，但对NVM而言，因掉电或故障重启后数据仍然存在，可能会导致程序执行的逻辑混乱。

任务内容：

1. 了解造成NVM不一致性的原因，调研已有的解决方案；

2. 提出维护NVM一致性的方案，证明其正确性并编码实现。

参考文献：

1）Y Jiang, H Chen, F Qin, C Xu, X Ma, J Lu. Crash consistency validation made easy. In Proceedings of the Symposium on the Foundations of Software Engineering (FSE), 133--143, 2016

2）T S Pillai, V Chidambaram, R Alagappan, S Al-Kiswany, A C Arpaci-Dusseau, R H Arpaci-Dusseau. All file systems are not created equal: On the complexity of crafting crash-consistent applications. In Proceedings of the Conference on Operating Systems Design and Implementation (OSDI), 433--448, 2014

3）Volos, H., Tack, A. J., and Swift, M. M. Mnemosyne: Lightweight persistent memory. In Proceedings of the Sixteenth International Conference on Architectural Support for Programming Languages and Operating Systems (2011), ASPLOS XVI, pp. 91–104

4）Giles, E. R., Doshi, K., and Varman, P. Softwrap: A lightweight framework for transactional support of storage class memory. In 2015 31st Symposium on Mass Storage Systems and Technologies (MSST) (2015), IEEE, pp. 1–14

5）Kolli,A.,Pelley,S.,Saidi,A.,Chen,P.M.,andWenisch,T.F.High-performance transactions for persistent memories. In Proceedings of the Twenty-First International Conference on Architectural Support for Programming Languages and Operating Systems (2016), ACM, pp. 399–411

6）Coburn, J., Caulfield, A. M., Akel, A., Grupp, L. M., Gupta, R. K., Jhala, R., and Swanson, S. NV-Heaps: making persistent objects fast and safe with next-generation, non-volatile memories. ACM Sigplan Notices 46, 3 (2011), 105–118

7）Mengxing Liu, Mingxing Zhang, Kang Chen, Xuehai Qian, Yongwei Wu, Weimin Zheng, and Jinglei Ren. 2018. DudeTx: Durable Transactions Made Decoupled. ACM Trans. Storage 1, 1 (January 2018), 29 pages

8）Jinyu Gu and Qianqian Yu and Xiayang Wang and Zhaoguo Wang and Binyu Zang and Haibing Guan and Haibo Chen. Pisces: A Scalable and Efficient Persistent Transactional Memory. 2019 USENIX Annual Technical Conference (USENIX ATC'19), 2019, pp. 913--928

把NVM当存储来用：文件系统的优化；

把NVM当内存来用：内核的checkpoint；

NVM的访问限制：擦写的优化（批量写）

NVM的文件系统的可扩展性；（甄艳洁）

[https://cn.bing.com/academic/profile?id=89a37603e4b51f1862486293075a1c2d&encoded=0&v=paper_preview&mkt=zh-cn&showCite=true](https://cn.bing.com/academic/profile?id=89a37603e4b51f1862486293075a1c2d&encoded=0&v=paper_preview&mkt=zh-cn&showCite=true)

[https://www.cs.purdue.edu/homes/hsu62/nvthreads-eurosys.pdf](https://www.cs.purdue.edu/homes/hsu62/nvthreads-eurosys.pdf)

NVthreads: Practical Persistence for Multi-threaded Applications

从系统的角度来优化：系统启动优化；（姜认为可以容易实现）

#### 后续工作


1. 姜老师：细化目标；
### 其它探索方向

#### RISCV H-mode Hypervisor

#### 实时及分布式OS

分布式的难点是不可靠通信

断点续传

### 自选题目

