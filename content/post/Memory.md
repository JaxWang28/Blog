
# ROM
Read Only Memory
只读，不能**随意更改**，断电保留。


Core Rope ROM 硬连接线
![](https://img.jacksonwang28.top/2024/08/c43da4d23c09ee4ac5b11d3d168ebce8.png)

https://hackaday.com/2013/10/09/making-a-core-rope-read-only-memory/



PROM 可编程只读存储的 ROM，允许一次性编程。PROM使用保险丝技术，通过高电压熔断特定的连线来存储数据。

![](https://img.jacksonwang28.top/2024/08/eecc0cac60f8a68462ce540586084f70.png)

https://www.eeeguide.com/programmable-read-only-memory-prom/






EPROM 可擦除可编程 RAM EPROM允许通过紫外线擦除内容并重新编程。EPROM通常带有透明窗，便于紫外线照射
![](https://img.jacksonwang28.top/2024/08/087a1d62a203b3eb23031a0e98f09a87.png)

https://hackaday.com/2018/01/17/improvising-an-eprom-eraser/


EEPROM（电可擦除可编程只读存储器）: EEPROM进一步发展，允许通过电流擦除和重写数据，极大提高了灵活性，成为早期嵌入式系统和微控制器中常用的存储器。






Flash Memory
一种与 EEPROM 相似的技术，关键不同点是**在闪存中只能写入整块单元内容，在写之前，这个块以前的内容被擦除，即先擦后写。**
FLASH属于广义上的ROM，和EEPROM的最大区别是FLASH按扇区操作，相对于EEPROM的改进就是擦除时不再以字节为单位，而是以块为单位，一次简化了电路，数据密度更高，降低了成本。上M的ROM一般都是FLASH。而EEPROM则按字节操作。

在过去的20年里，嵌入式系统一直使用 **ROM**（EPROM）作为它们的存储设备，然而近年来Flash全面代替了ROM（EPROM）在嵌入式系统中的地位，用作存储Bootloader以及操作系统或者程序代码或者直接当硬盘使用（U盘）。






# RAM
Random Access Memory

 随机存取存储器。是与 **CPU** 直接交换数据的内部存储器，也叫内存。它可以随时读写，而且速度很快，通常作为操作系统或其他正在运行中的程序的临时数据存储媒介, 当电源关闭时RAM不能保留数据。分为静态动态。



- **静态RAM**（Static RAM/SRAM）:SRAM速度非常快，不需要刷新电路即能保存数据，是目前读写最快的存储设备了，但是集成度较低，非常昂贵，多用于CPU的一级缓存，二级缓存(L1/L2 Cache)。只要不停止供电就能一直保持状态。访问速度非常之快。

静态随机存储器速度很快，但是它们的存储器件成本较多且占用空间。使用同等存储器的存储器件平面内可以实现更高的密度，更高密度的随机存储器件价格较高。但是，这些更简单的随机存储器件不可能实现例如缓存区的相关。除非它们连接经常访问的数据载体，处理这种情况的存储器速度较为缓慢的动态随机存储器（DRAM）

`DRAM` 分为很多种，常见的主要有 FPRAM/FastPage、EDORAM、SDRAM、DDR RAM、RDRAM、SGRAM 以及 WRAM 等，这里介绍其中的一种 DDR RAM。










DRAM保留数据的时间很短(需要内存刷新电路，每隔一段时间，刷新充电一次，否则数据会消失)，速度也比SRAM慢，不过它还是比任何的ROM都要快，但从价格上来说DRAM相比SRAM要便宜很多，计算机内存就是DRAM的。

SDRAM synchronous DRAM。
显著特点使用时钟信号，内置刷新电路。





`DDR RAM`（Date-Rate RAM）也称作 DDR SDRAM ，这种改进型的RAM和SDRAM是基本一样的，不同之处在于它可以在一个时钟读写两次数据，这样就使得数据传输速度加倍了。这是目前电脑中用得最多的内存，而且它有着成本优势，事实上击败了Intel的另外一种内存标准－Rambus DRAM。在很多高端的显卡上，也配备了高速 DDR RAM 来提高带宽，这可以大幅度提高3D加速卡的像素渲染能力。







# Flash Memory






https://gist.github.com/but0n/d773f855b1641e156f2d2e40ada9dbdc



https://bbs.elecfans.com/jishu_2240698_1_1.html