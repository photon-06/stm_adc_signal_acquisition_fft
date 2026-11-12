# 操作指南

## L0：CubeMX点灯 + 串口输出Hello

1. CubeMX配置：开启GPIO（LED）、USART1，配置时钟，生成Keil工程
2. Keil编译代码，下载到STM32板子
3. CH340接电脑，装好驱动，找到COM口
4. 打开串口助手，波特率115200
5. 上电：LED闪烁，串口持续打印 Hello

接线示意图：

!\[L0：CubeMX点灯 + 串口输出Hello 整体接线图](circuit/l0\_circuit.png)

\---

## L1：ADC单通道采集（轮询读取）

1. CubeMX配置：开启ADC1单通道，配置为独立模式、轮询采集；保持原有USART1、LED、时钟配置不变，重新生成Keil工程
2. 硬件接线：电位器两端分别接3.3V、GND，电位器中间抽头引脚接到STM32 ADC输入引脚
* 重点：电位器GND和STM32、CH340共地
3. 编写代码：轮询读取ADC数值，转换之后通过串口打印原始ADC采样值
4. Keil编译，下载固件
5. 打开串口助手，波特率115200
6. 旋转电位器，观察串口输出的数字跟随变化
7. 统一数据输出格式，固定串口输出样式，方便后续上位机解析

接线示意图：

!\[L1：ADC单通道采集 整体接线图](circuit/l1\_circuit.jpg)

\---

## L2：定时采集 + 连续数据流

1. CubeMX配置：

   * 开启TIM2：Prescaler填7199，Counter Period填99（10ms定时）。
   * NVIC Settings勾选TIM2 global interrupt。
   * ADC1的Continuous Conversion Mode改为Disable。
2. Keil代码：

   * BEGIN 2：启动 HAL\_TIM\_Base\_Start\_IT(\&htim2);
   * BEGIN 3：保持清空。
   * BEGIN 4：编写 HAL\_TIM\_PeriodElapsedCallback，在中断中读取ADC，拆分为高低8位，加帧头0xAA和异或校验，通过 HAL\_UART\_Transmit 发送4字节数组。
3. 验证：串口助手勾选“16进制显示”，观察到 AA XX XX XX 格式的稳定数据帧连续刷屏。

