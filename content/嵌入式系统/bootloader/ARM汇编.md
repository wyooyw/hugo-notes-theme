在线ARM模拟网站：https://cpulator.01xz.net/?sys=arm-de1soc

- mrs
	- **MRS Rd, psr**，作用： 把状态寄存器的值，赋值给通用寄存器 Rd
	- psr可以为CSPR,....
- msr：与mrs反着，将通用寄存器的值存到状态寄存器
- cpsr：当前程序状态寄存器，见后面
- teq：测试两个值是否相等。若不相等，会发生什么？
- bic：清除原寄存器中指定的位
- bince：看起来功能和bic相同
- orrne：按位或，存到目标寄存器中
- orr：看起来和orrne一样
- FIQ：Fast Imterrupt Request，快中断状态
- IRQ：Imterrupt Request，中断状态
- SVC：Supervisor，SVC模式是供操作系统使用的一种保护模式
- HYP：Hypervisor，HYP模式用于对虚拟化有关功能的支持
- 跳转指令：b


CSPR状态寄存器：
![](ARM汇编-1667287286350.jpeg)

