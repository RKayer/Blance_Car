# 平衡小车
## 电机
电机首先要有一个电机驱动模块，这里我用的是TT1661电机驱动模块
PWMA---连接程序中的端口    |     VM------连接高电平
|-|-|
AIN2-----连接高电平（1）  |     VCC-----连接VCC
AIN1-----连接低电平（0） |        GND-----接地
STBY----连接VCC   |   A01-----连接电机1(+)
BIN1---  连接低电平（0）    |   A02-----连接电机1(-)
BIN2---- 连接高电平（1）     |     B2------连接电机2
PWMB---连接程序中的端口   |             B1------连接电机2
GND-----接地     |                           GND----接地

![](https://gitee.com/RKayer/blogimage/raw/master/img/tt6612.png)

```c
	/*
     * STBY : 0 -> 电机驱动使能关 1 -> 电机使能开
	 * AIN1   0    0     1
	 * AIN2   0    1     0
	 *       停止 正转  反转
	 */
```

一个模块可以控制2个电机，分别对应A，B

## 编码器
![](https://gitee.com/RKayer/blogimage/raw/master/img/encoder.png)

编码器有A，B两相接口，连接到单片机的定时器两个输入捕获通道，一般来说接到同一个定时器的两个捕获通道。同时，定时器设置为编码器模式。定时器1，2，3，4，5和8有编码器的功能。
单片机会自动对A，B相脉冲(输出对应TI1和TI2）进行读取，同时计数器也在不断在ARR范围内进行向上或向下计数，我们只需要读取计数值便可以确定编码器对应的电机行驶的距离.同时也可以知道正反转，因为TI1和TI2的脉冲相位差会决定计数是向上还是向下，所以我们的计数值加减可以反应出正反转。
**计算速度**
我们可以读取计数值得知单位时间的计数差，就可以求出速度差。
```c
//单位时间进入一次函数，然后读取出编码器的计数值，然后清零计数器，计数值就等于速度大小
/*单位时间编码器计数 输入定时器 输出速度值*/
int Read_Encoder(u8 TIMX)
{
	int Encoder_TIM;    
	switch(TIMX)
	{
		case 2:  Encoder_TIM= (short)TIM2 -> CNT;  TIM2 -> CNT=0;break;
		case 3:  Encoder_TIM= (short)TIM3 -> CNT;  TIM3 -> CNT=0;break;	
		case 4:  Encoder_TIM= (short)TIM4 -> CNT;  TIM4 -> CNT=0;break;	
		default:  Encoder_TIM=0;
	}
	return Encoder_TIM;
}

```
[编码器详解](https://blog.csdn.net/weixin_44270218/article/details/113664633)

[编码器更多了解](https://blog.csdn.net/weixin_44270218/article/details/113665051)

## MPU6050

MPU6050内部整合了三轴MEMS陀螺仪、三轴MEMS加速度计以及一个可扩展的数字运动处理器DMP
模块里面包含了两个传感器
**陀螺仪传感器**
**加速度传感器**
mpu6050中我们用陀螺仪传感器测角度，用加速度传感器测加速度（具体原理参考大物转动惯量和角加速度知识）
**DMP**
![](https://gitee.com/RKayer/blogimage/raw/master/img/mpu1.png)
数字运动处理器，就是将上面两个传感器的ADC值整合运算处理，可以得到我们更需要的YAW，ROLL，PITCH角，这三个角可以反映出物体的状态。
这个模块通过IIC接口协议和外部设备通信。

一般只用四个引脚VCC,GND,SCL,SDA
INT是中断引脚
SCL--IIC时钟线，
SDA--IIC数据线
AD0:地址线，接3V地址为0x68，接地地址为0x69
![](https://gitee.com/RKayer/blogimage/raw/master/img/mpu2.png)

[MPU模块代码移植工程参照](https://blog.csdn.net/weixin_45523734/article/details/107318944?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163299025416780261945800%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163299025416780261945800&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-107318944.pc_search_result_cache&utm_term=mpu6050&spm=1018.2226.3001.4187)

## PID闭环控制
平衡车比较难的一个东西，PID算法理论之前学51单片机的了解一点，但是也没有自己实践过，也是一知半解，现在有个平衡PID控制算法，也想深入学习一下。
**P**--Proportion--比例
**I**--Integration--积分
**D**--Differentiation--微分

 PA6 PA7
 PB3 PB15

