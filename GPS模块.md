# arduino驱动ATGM332D
GPS模块VCC————–Arduino的5v  
GPS模块GND————–Arduino的GND  
GPS模块TXD————–Arduino的D0（数字IO 0）  
GPS模块RXD不接  
GPS模块PPS不接  
其中GPS的天线记得要接，并且要室外放置，因为GPS模块是室外定位的，室内没有卫星信号。在室外时需要大概1分钟左右的时间来获取当前位置数据。  
Arduino自测程序：  
```
void setup()    
{
  Serial.begin(9600);           //定义波特率9600
  Serial.println("Wating...");
}

void loop()     
{
  while (Serial.available()) {   
     Serial.write(Serial.read());//收到GPS数据则通过串口输出
  }
}
```
![image](https://github.com/xin1/under-water-robot/assets/81465751/cb779f8e-d52c-4ed1-bb28-868896552ade)

GPS数据

一组数据如下：
```
$GNGGA,,,,,,0,00,25.5,,,,,,*64
$GNGLL,,,,,,V,N*7A
$GPGSA,A,1,,,,,,,,,,,,,25.5,25.5,25.5*02
$BDGSA,A,1,,,,,,,,,,,,,25.5,25.5,25.5*13
$GPGSV,1,1,00*79
$BDGSV,1,1,00*68
$GNRMC,,V,,,,,,,,,,N*4D
$GNVTG,,,,,,,,,N*2E
$GNZDA,,,,,,*56
$GPTXT,01,01,01,ANTENNA OK*35

$GNGGA,084852.000,2236.9453,N,11408.4790,E,1,05,3.1,89.7,M,0.0,M,,*48
$GNGLL,2236.9453,N,11408.4790,E,084852.000,A,A*4C
$GPGSA,A,3,10,18,31,,,,,,,,,,6.3,3.1,5.4*3E
$BDGSA,A,3,06,07,,,,,,,,,,,6.3,3.1,5.4*24
$GPGSV,3,1,09,10,78,325,24,12,36,064,,14,26,307,,18,67,146,27*71
$GPGSV,3,2,09,21,15,188,,24,13,043,,25,55,119,,31,36,247,30*7F
$GPGSV,3,3,09,32,42,334,*43
$BDGSV,1,1,02,06,68,055,27,07,82,211,31*6A
$GNRMC,084852.000,A,2236.9453,N,11408.4790,E,0.53,292.44,141216,,,A*7
5
$GNVTG,292.44,T,,M,0.53,N,0.98,K,A*2D
$GNZDA,084852.000,14,12,2016,00,00*48
$GPTXT,01,01,01,ANTENNA OK*35
```
数据里面我们看到三种数据类型  
GN、GP、BD 分别代表双模模式、GPS 模式、北斗模式  
(1) $GPGGA （GPS 定位信息）  
GPGGA  
![image](https://github.com/xin1/under-water-robot/assets/81465751/8a9fde73-0d39-4b5a-9ca2-6e13c5750386)

(2) $GPGLL （地理定位信息）  
GPGLL  
![image](https://github.com/xin1/under-water-robot/assets/81465751/1b76c0b3-7ef0-4ff2-affc-20cfcf13cbc8)

(3)$GPGSA （当前卫星信息）  
GPGSA  
![image](https://github.com/xin1/under-water-robot/assets/81465751/5760ab52-811f-4d87-b583-d9c28bbc01a7)

(4) $GPGSV（可见卫星信息）  
GPGSV  
![image](https://github.com/xin1/under-water-robot/assets/81465751/25fe2be4-a937-4ab2-b266-d3b5f96cdcd9)

(5) $GPRMC（最简定位信息）这个可用信息较多  
GPRMC  
![image](https://github.com/xin1/under-water-robot/assets/81465751/4b8fda5a-e710-4f59-9de8-dfbae6173d12)

(6) $GPVTG（地面速度信息）  
GPVTG  
![image](https://github.com/xin1/under-water-robot/assets/81465751/008308b2-1b35-4e67-a44c-12fce7ddd8e4)

（7）天线状态输出  
$GPTXT,01,01,01,ANTENNA OK*35   
Ok 代表天线已经检测到，open 代表天线断开。  

关于热启动温启动冷启动的阐述  
冷启动是指在一个陌生的环境下启动GPS 直到GPS 和周围卫星联系并且计算出  
坐标的启动过程。以下几种情况开机均属冷启动：  
1、初次使用时；  
2、电池耗尽导致星历信息丢失时；  
3、关机状态下将接收机移动1000 公里以上距离。也就是说冷启动是通过硬件方
式的强制性启动，因为距离上次操作GPS 已经把内部的定位信息清除掉，GPS 接
收机失去卫星参数，或者已经存在的参数和实际接收到卫星参数相差太多，导致
导航仪无法工作，必须重新获得卫星提供的坐标数据，所以说车辆从地库里启动
导航百分百算冷启动，这也是从地库出来搜星时间长的原因。  
温启动是指距离上次定位时间超过2 个小时的启动，搜星定位时间介于冷启动和
热启动之间。如果您前一日使用过GPS 定位，那么次日的第一次启动就属于温启
动，启动后会显示上次的位置信息。因为上次关机前的经纬度和高度已知，但由
于关机时间过长，星历发生了变化，以前的卫星接受不到了，参数中的若干颗卫
星已经和GPS 接收机失去了联系，需要继续搜星补充位置信息，所以搜星的时间
要长于热启动，短于冷启动。  
热启动是指在上次关机的地方没有过多移动启动GPS，但距离上次定位时间必须
小于2  

GPS解析程序：  
```
#define GpsSerial  Serial
#define DebugSerial Serial
int L = 13; //LED指示灯引脚

struct
{
    char GPS_Buffer[80];
    bool isGetData;     //是否获取到GPS数据
    bool isParseData;   //是否解析完成
    char UTCTime[11];       //UTC时间
    char latitude[11];      //纬度
    char N_S[2];        //N/S
    char longitude[12];     //经度
    char E_W[2];        //E/W
    bool isUsefull;     //定位信息是否有效
} Save_Data;

const unsigned int gpsRxBufferLength = 600;
char gpsRxBuffer[gpsRxBufferLength];
unsigned int ii = 0;


void setup()    //初始化内容
{
    GpsSerial.begin(9600);          //定义波特率9600，和我们店铺的GPS模块输出的波特率一致
    DebugSerial.begin(9600);
    DebugSerial.println("ILoveMCU.taobao.com");
    DebugSerial.println("Wating...");

    Save_Data.isGetData = false;
    Save_Data.isParseData = false;
    Save_Data.isUsefull = false;
}

void loop()     //主循环
{
    gpsRead();  //获取GPS数据
    parseGpsBuffer();//解析GPS数据
    printGpsBuffer();//输出解析后的数据
    // DebugSerial.println("\r\n\r\nloop\r\n\r\n");
}

void errorLog(int num)
{
    DebugSerial.print("ERROR");
    DebugSerial.println(num);
    while (1)
    {
        digitalWrite(L, HIGH);
        delay(300);
        digitalWrite(L, LOW);
        delay(300);
    }
}

void printGpsBuffer()
{
    if (Save_Data.isParseData)
    {
        Save_Data.isParseData = false;

        DebugSerial.print("Save_Data.UTCTime = ");
        DebugSerial.println(Save_Data.UTCTime);

        if(Save_Data.isUsefull)
        {
            Save_Data.isUsefull = false;
            DebugSerial.print("Save_Data.latitude = ");
            DebugSerial.println(Save_Data.latitude);
            DebugSerial.print("Save_Data.N_S = ");
            DebugSerial.println(Save_Data.N_S);
            DebugSerial.print("Save_Data.longitude = ");
            DebugSerial.println(Save_Data.longitude);
            DebugSerial.print("Save_Data.E_W = ");
            DebugSerial.println(Save_Data.E_W);
        }
        else
        {
            DebugSerial.println("GPS DATA is not usefull!");
        }

    }
}

void parseGpsBuffer()
{
    char *subString;
    char *subStringNext;
    if (Save_Data.isGetData)
    {
        Save_Data.isGetData = false;
        DebugSerial.println("**************");
        DebugSerial.println(Save_Data.GPS_Buffer);


        for (int i = 0 ; i <= 6 ; i++)
        {
            if (i == 0)
            {
                if ((subString = strstr(Save_Data.GPS_Buffer, ",")) == NULL)
                    errorLog(1);    //解析错误
            }
            else
            {
                subString++;
                if ((subStringNext = strstr(subString, ",")) != NULL)
                {
                    char usefullBuffer[2]; 
                    switch(i)
                    {
                        case 1:memcpy(Save_Data.UTCTime, subString, subStringNext - subString);break;   //获取UTC时间
                        case 2:memcpy(usefullBuffer, subString, subStringNext - subString);break;   //获取UTC时间
                        case 3:memcpy(Save_Data.latitude, subString, subStringNext - subString);break;  //获取纬度信息
                        case 4:memcpy(Save_Data.N_S, subString, subStringNext - subString);break;   //获取N/S
                        case 5:memcpy(Save_Data.longitude, subString, subStringNext - subString);break; //获取纬度信息
                        case 6:memcpy(Save_Data.E_W, subString, subStringNext - subString);break;   //获取E/W

                        default:break;
                    }

                    subString = subStringNext;
                    Save_Data.isParseData = true;
                    if(usefullBuffer[0] == 'A')
                        Save_Data.isUsefull = true;
                    else if(usefullBuffer[0] == 'V')
                        Save_Data.isUsefull = false;

                }
                else
                {
                    errorLog(2);    //解析错误
                }
            }


        }
    }
}


void gpsRead() {
    while (GpsSerial.available())
    {
        gpsRxBuffer[ii++] = GpsSerial.read();
        if (ii == gpsRxBufferLength)clrGpsRxBuffer();
    }

    char* GPS_BufferHead;
    char* GPS_BufferTail;
    if ((GPS_BufferHead = strstr(gpsRxBuffer, "$GPRMC,")) != NULL || (GPS_BufferHead = strstr(gpsRxBuffer, "$GNRMC,")) != NULL )
    {
        if (((GPS_BufferTail = strstr(GPS_BufferHead, "\r\n")) != NULL) && (GPS_BufferTail > GPS_BufferHead))
        {
            memcpy(Save_Data.GPS_Buffer, GPS_BufferHead, GPS_BufferTail - GPS_BufferHead);
            Save_Data.isGetData = true;

            clrGpsRxBuffer();
        }
    }
}

void clrGpsRxBuffer(void)
{
    memset(gpsRxBuffer, 0, gpsRxBufferLength);      //清空
    ii = 0;
}
```

### 结果
![image](https://github.com/xin1/under-water-robot/assets/81465751/d199056b-3972-401c-8d0b-0c636428f1f5)
