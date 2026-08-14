 <img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/cover.png?raw=true" width="700"  /> 

## 项目简介
传统的智能手表大多依赖IMU（加速度计+陀螺仪）进行识别手势，无法捕捉手指的精细动作，且受手臂摆动干扰较大。本项目旨在开发一款基于表面肌电信号的智能手表，通过采集前臂肌肉群产生的生物电信号，实现高精度、低延迟的手势识别，进而配置到电脑的交互映射，为用户提供一种全新的、无接触的交互方式。

## 项目功能
- 功能描述：
  1. 使用佩戴在手腕上的八对差分电极的环形阵列来采集表面肌电；
  2. 实现拇指伸展、食指伸展、捏拳、手掌张开的手势识别；
  3. 板载低功耗无线透传电路，实现智能手表与电脑无线连接通信；
  4. 支持手势映射为键鼠指令，如“拇指伸展”=“上一页”，“食指伸展”=“下一页”，“捏拳”=“缩小”，“手掌张开”=“放大”；
  5. 搭载低功耗显示屏，用于显示手势激活状况、手势识别结果、电量详情等；
  6. 采用Type-C接口充电，内置锂电池供电使用。

- 实时手势识别演示视频 ([完整视频下载](https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_inference_demo.mp4))

 <img src="https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_inference_demo.gif" width="960" height="540"  /> 

- 实时表面肌电数据采集 ([完整视频下载](https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_sEMG_acqusition.mp4))

![acqusition](https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_sEMG_acqusition.gif)

- 实时OLED显示肌电激活 ([完整视频下载](https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_oled_plot.mp4))

![oled](https://github.com/BrianQiu1262/sEMG_Watch/raw/refs/heads/main/Video/realtime_oled_plot.gif)

## 项目系统框架

![system](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/System%20pipline.png?raw=true)
### 肌电手表
* 采用STM32L4低功耗系列作为主控芯片，控制ADS1299这一生理信号模拟前端芯片进行sEMG信号采集。采集速率为250Hz，信号分辨率为24bit；
* 将采集得到的信号，使用实时IIR滤波器依次进行5Hz二阶高通滤波、以及49-51Hz陷波，以去除直流漂移以及工频噪声；
* 利用双缓存填充数据帧、以及通过串口将数据传输给NRF52832；
* 采用NRF52832作为无线透传芯片，接收来自STM32的数据并将其发送给NRF接收器。同时接收从接收器侧返回的手势识别结果，并回传给STM32以进行OLED屏幕回显。
![system](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch%20hardware.png?raw=true)

### NRF接收器
* 采用NRF52832作为无线透传芯片，接收来自肌电手表的数据并将其发送给PC；
* 通过串口转USB模块将数据通过USB-TYPE-A接口传到PC。

<table>
  <tr>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Receiver_PCB.png?raw=true" alt="1" / width="800"></td>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Receiver_3D.png?raw=true" alt="2" / ></td>
  </tr>
</table>

### PC-C#上位机
* C#上位机通过串口接收无线透传数据，并实时进行8通道的sEMG绘图工作；
* 通过按钮进行8通道sEMG数据的采集与保存；
* 利用TCP协议与Python代码进行通信，将数据传输至Python并接收手势识别结果。

![](https://github.com/BrianQiu1262/sEMG_Watch_CSharp_Demo/blob/main/interface.png?raw=true)

### PC-Python代码
* Python利用C#采集并保存的数据集进行特征计算、模型训练、以及模型保存；
* 实现实时推理功能，将接收到的实时sEMG数据进行手势识别并将结果返回给C#上位机。

## 硬件说明
### 肌电手表
<table>
  <tr>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_PCB.jpg?raw=true" alt="1" /></td>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_3D_top.png?raw=true" alt="2" /></td>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_3D_bottom.png?raw=true" alt="2" /></td>
  </tr>
</table>

* 模拟前端原理图

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_SCH_AFE.png?raw=true)

1. 正负2.5V双电源供电；
2. 8对差分输入；
3. 右腿驱动RLD用来消除共模噪声。
4. 通过FPC连接器连接FPC电极阵列即可进行sEMG采集

* 电源原理图

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_SCH_Power.png?raw=true)

1. 通过USB-TYPE-C接口插入普通数据线即可充电（非苹果系插头），橙灯亮起表示正在充电，橙灯熄灭则表示充电完成；
2. 打开开关亮绿灯表示设备正常工作；
3. 电池电压先转换成+3.3V供给数字电路，+3.3V通过正负电源转换以及滤波后再供给模拟电路。

* 主控原理图

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Watch_SCH_MCU.png?raw=true)

1. USB-TYPE-C接口还能外接烧录器进行STM32以及NRF的芯片烧录；
2. 按键可以来回切换两种模式：默认为数据透传模式；按下按键可切换为OLED数据屏显模式，即数据不通过无线发送到PC，而是在本地进行8通道的平均激活程度计算，并可视化激活程度（见实时演示视频）；
3. 电池电压通过电阻分压接入STM32的ADC进行电池电量的换算；
4. OLED显示屏左上角为回传的手势识别结果，右上角为电池电量（0-100）。

<table>
  <tr>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/interface.jpg?raw=true" alt="1" /width="500"></td>
    <td><img src="https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/OLED.png?raw=true" alt="2" /></td>
  </tr>
</table>

### FPC环形电极阵列
1. 从左到右的上下两行依次为8对sEMG差分电极，上下为一对；
2. 中间一行为RLD右腿驱动电极，所有的电极都连通等电位；
3. 佩戴的时候绕腕部一周，通过胶布和皮筋进行固定。

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/FPC.png?raw=true)

### NRF接收器

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/Receiver_SCH.png?raw=true)

1. 将USB-TYPE-A接口插入电脑，NRF52832即可通过加长天线接收手表传过来的数据并透传至PC；
2. 插入电脑后亮绿灯表示设备正常工作；
4. 串口转USB芯片的输入处接入了指示灯，如有数据传输发生，从NRF传输到PC亮红灯，从PC返回NRF亮蓝灯（如数据量少可能较暗）。

## 软件代码
1. [STM32代码](https://github.com/BrianQiu1262/sEMG_Watch_STM32)
- 利用ADS1299的GPIO数据到达中断作为定时器进行整个流程控制；
- 屏幕刷新在数据透传模式的时候为手势结果和电量发生变化的时候才更新，而在屏显模式的时候更新率为50Hz；
- 发送数据帧共29 (5个功能字 + 24个数据字) 个字节，十六进制为AA AA F1 18 ... (8x3字节) xx。前三个字节为帧头，第四个字节为数据长度，然后依次3个字节为不同通道sEMG的24bit采集电压，最后一位为判断是否丢包的标志位，其数值从0增大到100然后循环往复；
2. [手表侧NRF52832发送代码](https://github.com/BrianQiu1262/sEMG_Watch_Wireless_TX)
3. [NRF接收器代码](https://github.com/BrianQiu1262/sEMG_Watch_Wireless_RX)
4. [C#上位机代码](https://github.com/BrianQiu1262/sEMG_Watch_CSharp_Demo)
- 数据采集时需要配置COM口以连接NRF接收器，在采集之前应等信号稳定之后再进行；
- C#若只完成数采工作的话可以不同Python代码一起运行；
- C#数采完成会保存成7列数据的csv格式文件，第一列为时间，后面8列为sEMG的电压值；
- C#实时推理时，0表示没有激活（即无手势状态），而1，2，3，4则代表对应的手势类别。
5. [Python推理代码](https://github.com/BrianQiu1262/sEMG_Watch_Python_Demo)
- 特征计算采用了sEMG的五种典型时域特征，依次是MAV（平均绝对值）、RMS（均方根）、WL（波形长度）、ZCR（过零率）、SSC(斜率符号变化);
- 识别模型使用了简单而计算速度快的线性分类器（训练时使用标准化与岭最小二乘）以便于实现实时低延迟的手势推理；

## 数采流程 （C#）
1. 将NRF连接器插入电脑，并使用VS studio运行C#代码，按照NRF连接器插入的COM（设备管理器）进行填写；
2. 点击Open按钮，把肌电手表绕腕部一周佩戴好，使用胶带和皮筋加固（提供有效正压力），再通过FPC连接器连接PCB，后打开开关，在NRF连接器上能看见红灯（接收器→PC）亮起，过一会8通道sEMG能够实时显示绘图；
3. 最开始未稳定的数据不要采集，待8通道sEMG基线稳定后便可以进行数据采集。开始采集时，按下Start按钮表示开始记录数据；
4. 结束则按Close按钮；

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/1.png?raw=true)

5. 在弹窗中输入文件名，接着csv格式的数据会保存到DATA文件夹下；

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/2.png?raw=true)

6. 依次按下Open→Start→Close就能进行依次数采。

## 模型训练流程 （Python）
1. 把采集得到的csv数据放入Python demo的/RAW/下
2. 运行data_preprocessing.py，绘制出sEMG波形图
`python data_preprocessing.py`

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/3.png?raw=true)

3. 根据绘制出来的8通道sEMG数据图挑选出手势激活的位置，并标定起始位置和结束位置，修改文件地址，再次运行程序将数据按照手势存入到/DATA/下等待训练；
4. 运行model_train.py, 对存入/DATA/下的数据进行训练，并将模型保存至/MODEL/下；
`python model_train.py --demo --interval 0.1`

## 实时推理流程（C# + Python）
1. 将NRF连接器插入电脑，使用VS studio运行C#代码，按照NRF连接器插入的COM（设备管理器）进行填写，暂时无需点击Open按钮；
2. 打开Python demo，运行inference_demo.py；
`python inference_demo.py`
3. Python命令行显示sokect初始化成功；
4. 点击C#窗口里面的Open按钮；
5. Python命令行会显示TCP连接成功；

![](https://github.com/BrianQiu1262/sEMG_Watch/blob/main/Image/4.png?raw=true)

6. 肌电手表佩戴稳妥后，打开开关，此时可以看到C#里面的手势结果发生了变化，实时推理成功；
7. 若需要控制键鼠，则按下HMI按钮，再鼠标选定PPT，之后就能通过手势控制PPT了。
