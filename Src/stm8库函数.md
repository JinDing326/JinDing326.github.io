# STM8库函数自学笔记

## 1. GPIO的应用

### 1.1 GPIO的模式介绍

*****

#### 1. 输入模式：

- **浮空输入（Input Floating）**
  
    >浮空输入在 I\O 模式中带有 IN_FL 字眼，如：GPIO_MODE_IN_FL_NO_IT 、GPIO_MODE_IN_FL_IT。
    >浮空输入也叫悬浮输入，一般把浮空输入和上拉输入做类比学习。浮空输入的电平不确定的，即使外部的一个很小的输入信号都会使其发生改变。如果引脚设置为悬空的情况下，读取该端口的电平是不确定的,一般做AD采样转换使用。  
    
- **上拉输入（Input pull-up）**
  
    >上拉输入在 I\O 模式中带有 IN_PU字眼，如：GPIO_MODE_IN_PU_NO_IT 、GPIO_MODE_IN_PU_IT。
    >上拉输入的时候，引脚内部有一个上拉电阻通过开关连接到电源VDD， 当引脚没有和外部电路连接时，设置上拉输入方式的I/O引脚电平是确定的高电平。

******

#### 2. 输出模式：
- **开漏输出（Output open-drain）**
  
    >开漏输出就是不输出电压，控制输出低电平时引脚接地，控制输出高电平时引脚既不输出高电平，也不输出低电平，为高阻态。如果外接上拉电阻，则在输出高电平时电压会拉到上拉电阻的电源电压。这种方式适合在连接的外设电压比单片机电压低的时候。输出端出跟集电极开路十分相似，工作原理也是一样的。不同的是，开漏输出使用的场效应管，使用时要加上拉电阻。

- **推挽输出（Output push-pull）**

    >推挽输出可以输出高低电平，连接数字器件；推挽结构一般指两个三极管分别受两互补信号的控股，总是在一个三极管导通的时候另一个截止。高低电平由IC的电源决定。

*****

#### 3. 中断与输出速度
>中断在I/O模式中带有IT字眼。中断只存在I/O输入中，因为在输出中设置中断没有任何意义。中断的意思就是中止当前的工作，然后去执行另外的任务，执行完之后再回来执行原来的任务。

>输出速度也只有存在I/O输出中，可以调整I/O的输出速度来将它们进行等级划分，如：low level,10MHz、high-impedance level,10MHz、high level, 10MHz、high-impedance level,2MHz、high level,2MHz.

*****

#### 4. GPIO初始化

>初始化GPIO时候，会有一个初始化电平的操作，例如GPIO_MODE_OUT_OD_LOW_FAST，GPIO_MODE_OUT_OD_HIZ_FAST，GPIO_MODE_OUT_PP_HIGH_FAST中，含有**LOW**，**HIZ**，**HIGH**，分别为**低电平**、**高阻抗电平**、**高电平**三种状态。复位之后，所有的引脚都是悬浮输入模式。

*****

### 1.2 外部中断输入

*****
#### 1. 允许外部中断的输入模式   
>GPIO_MODE_IN_FL_IT 	带中断的浮空输入
>GPIO_MODE_IN_PU_IT  	带中断的上拉输入

*****

#### 2. 外部中断管理函数

>`void EXTI_DeInit  ( void   )` //将外部中断控制寄存器初始化为默认值。
>
>`XTI_GetExtIntSensitivity  ( EXTI_Port_TypeDef  Port )` //指定端口的中断触发方式 
>
>`EXTI_GetTLISensitivity  ( void   ) ` //获取高级中断触发方式
>
>`void EXTI_SetExtIntSensitivity  ( EXTI_Port_TypeDef  Port,  EXTI_Sensitivity_TypeDef  SensitivityValue  ) ` //设置指定端口的外部中断触发方式
    >
    >`void EXTI_SetTLISensitivity  ( EXTI_TLISensitivity_TypeDef  SensitivityValue )  ` //设置/获取高级中断触发方式

*****

#### 3. 中断映射表  

<img src="https://img.it610.com/image/info5/d219cd46fb1b4e85ad5ef6c3626365e4.jpg" align=center />

- IAR使用**寄存器方式**编写中断服务函数，需要进入`isostm8sxxx.h`查看中断向量编号  
    >例如：
    >`#define AWU_vector          3 /* IRQ No. in STM8 manual:  1 */`
    >`#define CLK_CSS_vector      4 /* IRQ No. in STM8 manual:  2 */`
    >`#define CLK_SWITCH_vector   4 /* IRQ No. in STM8 manual:  2 */`
    >`#define EXTI0_vector        5 /* IRQ No. in STM8 manual:  3 */`
    >
    >中断服务函数格式：
    >`#pragma vector =3  //AWU唤醒中断编号=中断向量号+2
    >__interrupt void GPIOD_IRQHandler(void)
    >{
    >	.......
    >}`  
    >                  
- IAR使用**库函数方式**编写中断服务函数，直接在`stm8s_it.c`文件中找到需要编写代码的位置  
    >例如：
    >Port_A的中断服务函数 直接在{..}里写需要进行的操作
    >`INTERRUPT_HANDLER(EXTI_PORTA_IRQHandler, 3)  //3——映射表中中断向量号
    >{
    >.....
    >}`

***思考***：  

1. 无论是寄存器方式还是库函数方式，中断服务函数的中断向量号都没指向地址，那是如何向地址进行赋值的，是在头文件里替换还是IAR的编译器进行设置？  





1. 关于中断服务函数中使用的GPIO_ReadInputPIN()函数会出现0(RESET)和1(SET)之外的值 ？
    >`if(GPIO_ReadInputPin(GPIOA,GPIO_PIN_3) !=RESET)
    >{
    >GPIO_WriteReverse (GPIOD,GPIO_PIN_2);
    >}`

    >解析：
    >`BitStatus GPIO_ReadInputPin  ( GPIO_TypeDef *  GPIOx, GPIO_Pin_TypeDef  GPIO_Pin  ) `
    >函数原型
    >`BitStatus GPIO_ReadInputPin(GPIO_TypeDef* GPIOx, GPIO_Pin_TypeDef GPIO_Pin)
    >{
    >return ((BitStatus)(GPIOx->IDR & (uint8_t)GPIO_Pin));
    >}`
    >返回输入引脚的状态，返回值为BitStatus类型
    >
    >BitStatus原型
    >`typedef enum {RESET = 0, SET = !RESET} FlagStatus, ITStatus, BitStatus, BitAction;`

    >例如PA3引脚设置为输入引脚，输入高电平时GPIOA->IDR=0x08，GPIO_Pin=(GPIO_PIN_3)0x08，GPIOA->IDR & GPIO_Pin=1，这样一看确实没有问题，但是STM8S的PA1，PA2是外部晶振接口，默认为输入，那此时GPIOA->IDR=0x0E，GPIOA->IDR & GPIO_Pin=0x08，出现了0和1之外的数，所以需要这样使用。
    >输入低电平，读取引脚状态：
    >`GPIO_ReadInputPin  ( GPIO_TypeDef *  GPIOx, GPIO_Pin_TypeDef  GPIO_Pin  )==RESET`
    >输入高电平，读取引脚状态：
    >
    >`GPIO_ReadInputPin  ( GPIO_TypeDef *  GPIOx, GPIO_Pin_TypeDef  GPIO_Pin  ) !=RESET` // !=  判断两边的值是否相等，相等为true

*****

### 1.3无源蜂鸣器控制

*****

#### 1.常用时钟函数
>`void CLK_SYSCLKConfig  ( CLK_Prescaler_TypeDef  CLK_Prescaler ) `//配置HSI和CPU时钟分频器

- 时钟分频系数
>`CLK_PRESCALER_HSIDIV1`  //High speed internal clock prescaler: 1  主时钟源分频系数，设置后主时钟源的频率为HSI RC时钟的1分频，即f~HSI~/1

>`CLK_PRESCALER_HSIDIV2`  High speed internal clock prescaler: 2 

>`CLK_PRESCALER_HSIDIV4`  High speed internal clock prescaler: 4 	

>`CLK_PRESCALER_HSIDIV8`  High speed internal clock prescaler: 8 

>`CLK_PRESCALER_CPUDIV1`  // CPU clock division factors 1 CPU时钟源分频系数，设置后CPU时钟源的频率为主时钟（f~MASTER~）的1分频，即f~CPU~/1	f~CPU~为CPU和窗口看门狗提供时钟信号

>`CLK_PRESCALER_CPUDIV2`  CPU clock division factors 2 

>`CLK_PRESCALER_CPUDIV4`  CPU clock division factors 4

>`CLK_PRESCALER_CPUDIV8`  CPU clock division factors 8

>`CLK_PRESCALER_CPUDIV16`  CPU clock division factors 16

>`CLK_PRESCALER_CPUDIV32`  CPU clock division factors 32

>`CLK_PRESCALER_CPUDIV64`  CPU clock division factors 64

>`CLK_PRESCALER_CPUDIV128`  CPU clock division factors 128 

*****

#### 2.无源蜂鸣器库函数
>`void BEEP_Cmd  ( FunctionalState  NewState )`   //启用或禁用BEEP功能。
>`void BEEP_DeInit  ( void   )`  //将BEEP外设寄存器初始化为默认值。
>`void BEEP_Init  ( BEEP_Frequency_TypeDef  BEEP_Frequency )`  //根据指定的参数初始化BEEP功能。
>`void BEEP_LSICalibrationConfig  ( uint32_t  LSIFreqHz )`  //用测得的LSI频率更新CSR寄存器。

- FunctionalState  NewState：
>`DISABLE`
>`ENABLE`

- BEEP_Frequency_TypeDef  BEEP_Frequency：
>`BEEP_FREQUENCY_1KHZ`  //Beep signal output frequency equals to 1 KHz  设置蜂鸣器输出1KHz频率
>`BEEP_FREQUENCY_2KHZ`  //Beep signal output frequency equals to 2 KHz 
>`BEEP_FREQUENCY_4KHZ` // Beep signal output frequency equals to 4 KHz 

*****

##  2. 定时器的应用

### 2.1 定时器介绍

****

#### 1.定时器/计时器的本质
>定时器的本质是一个**计数器**，当计数的**脉冲频率一定**时，也就是说计数时钟的**周期一定**，获取脉冲的个数与时钟周期的乘积得到具体的定时时间，此时定时器表现为**定时功能**；当计数的**脉冲频率不固定**时，**周期也就不固定**，只能得到脉冲的个数，此时定时器表现为**计数功能**。

****

#### 2.TIM4




