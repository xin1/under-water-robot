# 通信功能
## 功能简介
索尼游戏控制器有12个对压力敏感的模拟键（4个方向键、4个操作键、十字、三角、圆形和方形，L1、L2、R1和R2）和5个数字键（MODE，START，SELECT， R3，L3）和2个模拟操纵杆。
![image](https://github.com/xin1/under-water-robot/assets/81465751/ebefb834-0d53-4ec0-8590-60b1de4a59b0)

无线控制器工作频率为2.4GHz，射程为10米。它还有一个用于发送和接收数据的光学指示器。该控制器仅需3节AAA电池供电（在某些情况下，它只需要2节AAA电池）。  
## 电路连接
PS2控制器接收器包括9个引脚：
![image](https://github.com/xin1/under-water-robot/assets/81465751/fcbae24a-b0f9-4725-9248-db7b97f651b2)

1. Data：主机线，用于向从站发送数据（MOSI）
2. Command：从机线，用于向主站发送数据（MISO）
3. Vibration：振动电机的电源; 7.2V至9V
4. Ground：电路接地
5. VCC：电源供电; 3.3伏
6. Attention：CS或芯片选择引脚用于调用从机并准备连接
7. Clock：相当于时钟的SCK引脚
8. No Connection：没用
9. Acknowledge：从控制器到PS2接收器的应答信号

连接PS2控制器和Arduino
为了使用PS2控制器，您需要将控制器的按键引入Arduino。然后根据您的项目为每个键选择适当的功能。
![image](https://github.com/xin1/under-water-robot/assets/81465751/7b9d446d-b268-48d2-b9cb-212dae1651ee)

## 代码
首先安装PS2X的库，库里搜不到可以下载本文文件夹PS2X_lib  放到Arduino\libraries路径下
以下代码可实现与PlayStation 2游戏手柄进行通信的功能。它包括配置控制器、读取按键状态、检测按键按下和释放事件以及读取摇杆的数值。
```
#include <PS2X_lib.h>

PS2X ps2x;

// 当前库不支持热插拔控制器，意味着连接控制器后必须重新启动Arduino，
// 或在连接控制器后再次调用config_gamepad(pins)函数。

int error = 0;
byte type = 0;
byte vibrate = 0;

void setup() {
  Serial.begin(57600);

  error = ps2x.config_gamepad(13, 11, 10, 12, true, true);   //GamePad(clock, command, attention, data, Pressures?, Rumble?)

  if (error == 0) {
    Serial.println("找到控制器，配置成功");
    Serial.println("尝试所有按钮，X按钮按下时手柄会震动，按下越重震动越快；");
    Serial.println("按住L1或R1会打印出模拟摇杆的值。");
    Serial.println("访问 www.billporter.info 获取更新和报告错误。");
  } else if (error == 1)
    Serial.println("未找到控制器，请检查连接，参见readme.txt启用调试。访问 www.billporter.info 获取故障排除提示");
  else if (error == 2)
    Serial.println("找到控制器但不接受命令。参见readme.txt启用调试。访问 www.billporter.info 获取故障排除提示");
  else if (error == 3)
    Serial.println("控制器拒绝进入Pressures模式，可能不支持。");

  type = ps2x.readType();
  switch (type) {
    case 0:
      Serial.println("未知控制器类型");
      break;
    case 1:
      Serial.println("找到DualShock控制器");
      break;
    case 2:
      Serial.println("找到GuitarHero控制器");
      break;
  }
}

void loop() {
  /* 必须读取游戏手柄以获取新值
     读取游戏手柄并设置振动值
     ps2x.read_gamepad(small motor on/off, larger motor strenght from 0-255)
     如果不启用振动，请使用ps2x.read_gamepad();不带参数
     
     你应该每秒至少调用一次
  */

  if (error == 1)
    return;

  if (type == 2) {

    ps2x.read_gamepad();          //读取控制器

    if (ps2x.ButtonPressed(GREEN_FRET))
      Serial.println("按下绿色按钮");
    if (ps2x.ButtonPressed(RED_FRET))
      Serial.println("按下红色按钮");
    if (ps2x.ButtonPressed(YELLOW_FRET))
      Serial.println("按下黄色按钮");
    if (ps2x.ButtonPressed(BLUE_FRET))
      Serial.println("按下蓝色按钮");
    if (ps2x.ButtonPressed(ORANGE_FRET))
      Serial.println("按下橙色按钮");

    if (ps2x.ButtonPressed(STAR_POWER))
      Serial.println("Star Power命令");

    if (ps2x.Button(UP_STRUM))          //按钮按下时为TRUE
      Serial.println("上弦");
    if (ps2x.Button(DOWN_STRUM))
      Serial.println("下弦");

    if (ps2x.Button(PSB_START))                   //按钮按下时为TRUE
      Serial.println("按住Start");
    if (ps2x.Button(PSB_SELECT))
      Serial.println("按住Select");

    if (ps2x.Button(ORANGE_FRET)) // 如果为TRUE，则打印摇杆值
    {
      Serial.print("Whammy Bar位置:");
      Serial.println(ps2x.Analog(WHAMMY_BAR), DEC);
    }
  } else { // DualShock控制器

    ps2x.read_gamepad(false, vibrate);          //读取控制器并将大电机设定为以'vibrate'速度旋转

    if (ps2x.Button(PSB_START))                   //按钮按下时为TRUE
      Serial.println("按住Start");
    if (ps2x.Button(PSB_SELECT))
      Serial.println("按住Select");

    if (ps2x.Button(PSB_PAD_UP)) {         //按钮按下时为TRUE
      Serial.print("上方向按住程度：");
      Serial.println(ps2x.Analog(PSAB_PAD_UP), DEC);
    }
    if (ps2x.Button(PSB_PAD_RIGHT)) {
      Serial.print("右方向按住程度：");
      Serial.println(ps2x.Analog(PSAB_PAD_RIGHT), DEC);
    }
    if (ps2x.Button(PSB_PAD_LEFT)) {
      Serial.print("左方向按住程度：");
      Serial.println(ps2x.Analog(PSAB_PAD_LEFT), DEC);
    }
    if (ps2x.Button(PSB_PAD_DOWN)) {
      Serial.print("下方向按住程度：");
      Serial.println(ps2x.Analog(PSAB_PAD_DOWN), DEC);
    }

    vibrate = ps2x.Analog(PSAB_BLUE);        //这将根据按下蓝色（X）按钮的力度设置大电机的振动速度

    if (ps2x.NewButtonState())               //如果任何按钮的状态发生变化（从按下到释放，或从释放到按下），则为TRUE
    {
      if (ps2x.Button(PSB_L3))
        Serial.println("按下L3");
      if (ps2x.Button(PSB_R3))
        Serial.println("按下R3");
      if (ps2x.Button(PSB_L2))
        Serial.println("按下L2");
      if (ps2x.Button(PSB_R2))
        Serial.println("按下R2");
      if (ps2x.Button(PSB_GREEN))
        Serial.println("按下Triangle");
    }

    if (ps2x.ButtonPressed(PSB_RED))             //如果刚刚按下按钮，则为TRUE
      Serial.println("刚刚按下Circle");

    if (ps2x.ButtonReleased(PSB_PINK))             //如果刚刚释放按钮，则为TRUE
      Serial.println("刚刚释放Square");

    if (ps2x.NewButtonState(PSB_BLUE))            // 如果按下按钮PSB_BLUE，则为TRUE
      Serial.println("X刚刚改变");

    if (ps2x.Button(PSB_L1) || ps2x.Button(PSB_R1)) // 如果其中任一按钮为TRUE，则打印摇杆值
    {
      Serial.print("摇杆数值:");
      Serial.print(ps2x.Analog(PSS_LY), DEC); // 左摇杆，Y轴。其他选项: LX、RY、RX
      Serial.print(",");
      Serial.print(ps2x.Analog(PSS_LX), DEC);
      Serial.print(",");
      Serial.print(ps2x.Analog(PSS_RY), DEC);
      Serial.print(",");
      Serial.println(ps2x.Analog(PSS_RX), DEC);
    }

  }

  delay(50);

}
```
