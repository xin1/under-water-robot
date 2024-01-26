# arduino控制两个玩具电机转动
![e62c2809c49c1078230a48aa935af27d](https://github.com/xin1/under-water-robot/assets/81465751/8d8efdbd-0f45-49fc-a31e-0b92bac82fbf)
### 参考
<img width="600" alt="image" src="https://github.com/xin1/under-water-robot/assets/81465751/21c1c420-5fd8-4da2-ac1a-1d71ba4d7411">
<img width="600" alt="image" src="https://github.com/xin1/under-water-robot/assets/81465751/a9443702-fb1c-4ce9-9c28-fc5eeb5fa920">

```
/* XY-2.5AD-Demo
 *  太极创客 www.taichi-maker.com
 *  2018-08-02
 *  
 * 通过串行通讯使用XY-2.5AD控制两个DC电机
 * 通过digitalWrite HIGH LOW 控制电机运行和停止。
 * 
 * 如果需要获取更多有关XY-2.5AD控制电机的相关知识，请前往太极创客网站
 * www.taichi-maker.com
 * 
 * XY-2.5AD 控制电机简介
 * 
 * DC电机     运行状态    IN1    IN2    IN3     IN4
 * 电机A     正转（调速） 1/PWM   0                
 * 电机A     反转（调速） 0     1/PWM            
 * 空转                   0       0
 * 刹车                   1       1
 * 电机B     正转（调速）               1/PWM    0
 * 电机B     反转（调速）                 0     1/PWM
 * 空转                                   0      0
 * 刹车                                   1      1
 * This example code is in the public domain.
*/
 
// XY-2.5AD 连接Arduino引脚编号
int IN1 = 3;
int IN2 = 5;
int IN3 = 6;
int IN4 = 9;
 
int pinNum;             //  控制引脚号
int ctrlVal;            //  电机运行控制
  
void setup() {
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  
  Serial.begin(9600);
  Serial.println("++++++++++++++++++++++++++++++");     
  Serial.println("+ Taichi-Maker XY-2.5AD Demo +");   
  Serial.println("+    www.taichi-maker.com    +");  
  Serial.println("++++++++++++++++++++++++++++++");   
}
  
void loop() {
  if (Serial.available()) {     // 检查串口缓存是否有数据等待传输 
    char cmd = Serial.read();   // 获取电机指令中电机编号信息      
    
    switch(cmd){ 
      case 'p':   // 设置引脚编号
        pinNum = Serial.parseInt();
        Serial.print("Pin Number ");
        Serial.print(pinNum);
        Serial.print(" ,");
        break;                 
        
      case 'a':   // 模拟模式控制电机
        ctrlVal = Serial.parseInt();
        analogWrite(pinNum, ctrlVal);
        Serial.print("Set Value ");
        Serial.print(ctrlVal);
        Serial.println(".");          
        break;    
 
      case 'd':   // 数字模式控制电机
        ctrlVal = Serial.parseInt();
        digitalWrite(pinNum, ctrlVal);
        Serial.print("Set Value ");
        Serial.print(ctrlVal);
        Serial.println(".");             
        break;   
 
      default:   // 未知指令
        Serial.println("Unknown Command");     
        break;  
    }
  }
}
```
