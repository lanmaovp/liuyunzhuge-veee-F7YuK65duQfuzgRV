
### 1\. OB、FB、FC功能


**OB（组织块）**：用于执行特定的任务（CPU启动、循环扫描、时间中断、硬件中断等），每个OB块均有一个特定的功能和优先级（特定事件发生时被触发）。
**FB（功能块）**：具有内部存储器，可保留状态数据，适用于多频次调用需要保持数据状态的场合（PID控制器、计数器等）。
**FC（功能）**：不保存任何内部状态数据，用于执行特定的功能，每次调用数据不保留（简单计算、逻辑操作）


**调用规则：**


调用是指在一个块中使用另一个程序块的功能，可以将一个任务拆分成更小、更易管理的部分，递归地使用其他程序块，每个部分由一个独立的FB或FC来处理。（嵌套深度为8层）


* OB为最高层次的程序块，内部可编写逻辑，调用其他FB、FC块；
* FB块可调用FB、FC，可利用其他功能块或功能完成复杂逻辑（临时变量并不存储在背景数据块中，只用于一个循环）；
* FC块可调用FB、FC，它自己不保存任何数据，（必须给所有形参分配实参）。


**说明：**


* 嵌套调用从循环OB开始，最多嵌套16层FB或FC；中断OB最多嵌套6层；安全程序嵌套只有4层
* 结构 (STRUCT) 和 PLC 数据类型 (UDT) 的嵌套深度为 8 级。具体嵌套深度取决于所使用的 CPU。


**OB的事件与优先级：**
OB的执行先后与其优先级有关，优先级高的OB会中断低级的OB。当CPU在执行主循环程序时，如有有一个高优先级的中断OB被触发时，CPU会暂停主程序，先执行中断OB，当其执行完成后继续主程序。
![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241205003118247-2070700593.png)




---


### 2\. 背景数据块是什么


函数块（FB）的调用称为实例。实例使用的数据存储在背景数据块中（实例寻址）。


* 要对来自当前块接口的变量进行寻址： \#\<变量名称\>
* 访问多重背景块的数据：\#\<多重实例名称\>.\<变量名称\>


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241205003702836-348385977.png)




---


### 3\. 优化与标准访问的区别


STEP 7 提供具有不同访问选项的数据块：


* 可优化访问的数据块 (S7\-1200/S7\-1500\)
* 可标准访问的数据块 (S7\-300 / S7\-400 / S7\-1200 / S7\-1500\)


**可优化访问的数据块**（符号寻址）


* 可优化访问的数据块没有固定的定义结构。在声明中，仅为数据元素分配一个符号名称，而不分配在块中的固定地址。
* 对于符号寻址，可以使用数据块的名称和变量名，并用圆点分隔。数据块的名称用引号括起来。
* 提高存储空间的使用率
* 提高 CPU 的性能
**注意事项：**
* 使用符号表示，少量的注释即可


**可标准访问的数据块**（绝对寻址）


* 可标准访问的数据块具有固定的结构。数据元素在声明中分配了一个符号名，并且在块中有固定地址。
* 对于绝对寻址，可以使用数据块的编号和数据块变量的绝对地址，并用圆点分隔。地址标识符 % 被自动设置为绝对地址的前缀




---


### 4\. 编程中的操作数


* PLC 变量
* 常量
* 背景数据块中的变量
* 全局数据块中的变量




---


### 5\. 片段寻址


可以选择包含所声明变量的特定地址区域，访问宽为 1 位、8 位、16 位或 32 位的区域。
![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241205010709381-1403221648.jpg)




---


### 6\. 间接寻址


在运行之前不计算地址的操作数，采用间接寻址方式时，单循环中各程序段可多次执行，而且每次运行时使用的操作数不同。


**间接寻址选项：**


* 通过指针间接寻址
* ARRAY 元素的间接索引
* 通过 DB\_ANY 数据类型间接寻址数据块。


**间接寻址示例1：**


* 使用下标访问来自不同的三个变量。




| 下标 | 变量 |
| --- | --- |
| 1 | Input\_WORD\_0——Word |
| 2 | Temperature——DInt |
| 3 | Output\_WORD\_4——Word |


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241206002125875-2029326102.png)


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241206001516283-1251849667.png)


根据在 Index 参数中指定的编号（1、2 或 3），执行“FC\_AccessGroupInt”指令的第一、第二或第三种情况。


**ARRAY 间接寻址示例2：**


* 使用下标访问将ARRAY三维数组的元素复制到 "MyTarget" 变量中。


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241206003112536-3345790.png)




---


### 7\. AT覆盖变量（访问变量内部的片段）


附加“AT”关键字声明来覆盖已经定义了的变量；例如：对WORD中内部的Byte进行寻址；


**使用注意事项：**


* 在标准访问块内（去掉“优化的块访问”）
* 可小于等于被覆盖变量的宽度


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241206223652016-595294600.png)




---


### 8\.PLC 数据类型（UDT）


* 使用 PLC 数据类型作为定义变量的基本数据类型。
* 作为创建全局数据块的模板。
* 更改会在所有的使用点自动更新。


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241206225702953-305895805.png)




---


### 9\.使用 IEC 定时器和计数器


**IEC 定时器和计数器的优势**


IEC 定时器和计数器的统一应用，可显著提高程序代码的运行效率。
采用这种方式具有以下优势：


* 通过新生成的背景数据块，可对块进行多次调用。
* IEC 计数器的计数范围更大。
* 与 S5 定时器相比，IEC 定时器性能更佳，且计时更为准确。


**应用的两种方法**
【1】TON IEC 定时器和 CTU IEC 计数器的数据声明为局部变量（应用于块接口中）。
【2】创建一个 ARRAY of ICE... 类型的全局数据块，并将变量元素拖放到程序段中


![](https://img2024.cnblogs.com/blog/3497053/202412/3497053-20241203020742898-119654877.png)




---


### 10\. 写程序时必备的OB块


**程序必备块（开发常用的OB块）**


* 主程序循环块OB1‌：这是用户程序的核心，负责协调整个程序的运行。OB1在CPU启动后循环执行，用户可以在其中调用其他用户块（如FB、SFB、FC、SFC等）‌
* ‌初始化程序块OB100‌：在CPU上电时调用一次，通常用于初始化设置‌
* 循环中断块OB30：用于处理需要定期执行的任务，如通讯处理等‌


**‌错误处理块‌（添加后可直接下载空程序）：**


* ‌时间错误块OB80‌：防止CPU扫描时间超过最大值‌
* ‌‌电源故障块OB81‌：防止后备电源失效导致CPU停机‌
* ‌诊断中断块OB82‌：通讯时需要下载‌
* ‌CPU硬件故障块OB84‌：处理CPU硬件故障导致的停机‌
* ‌通信错误块OB87‌：处理通信错误‌‌
* ‌编程错误块OB121‌：处理编程错误‌
* ‌‌I/O错误块OB122‌：处理I/O错误‌
* ‌冗余错误块OB72‌：处理冗余系统错误‌‌（冗余系统时使用）




---


* ‌日期中断块OB10\~OB17‌：根据设定的日期和时间触发执行，适用于需要按日期执行的任务‌
* ‌时间延迟中断块OB20\~OB23‌：按设定的延迟时间触发执行，适用于需要延迟执行的任务‌
* ‌硬件中断块OB40\~OB47‌：由外部设备触发，适用于处理硬件相关的事件‌（硬件中断模块HF使用）




---


**经验1：**FC函数中的置复位指令无法执行？


* FC函数中置复位使用需要将其变量属性安排到IN\_OUT内。使用Input、output、Temp变量无法保持！


![](https://img2024.cnblogs.com/blog/3497053/202410/3497053-20241018212306182-1510387152.png)




---


# 待续


 本博客参考[Flowercloud 机场订阅加速](https://flowercloud6.com)。转载请注明出处！
