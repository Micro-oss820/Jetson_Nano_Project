# Jetson_Nano_Project

## Supplies for a project

### 1. JetsonNano
![KakaoTalk_20241219_195505659_01](https://github.com/user-attachments/assets/f330eeec-3e65-4dcf-930a-e8337a67c4da)
### 2. Arduino, Sensor : CM1106, Grove dust sensor
![KakaoTalk_20241219_195505659](https://github.com/user-attachments/assets/74ef87ed-2d30-49b3-8ba1-9fe869cce487)

## Install Arduino

### 1. Download version 1.8.19 from the Home Page : https://www.arduino.cc/en/software

### 2. Unzip Arduino files : 

cd Downloads
tar -xf arduino-1.8.19-linuxaarch64.tar.xz 
cd arduino-1.8.19

### 3. Arduino Installation/Setup : 

sudo ./install.sh
sudo usermod -aG dialout $USER
newgrp dialout

arduino (→ Practice)

## Code execute 1

#include <SoftwareSerial.h>
 
SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11,0x01,0x01,0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;
 
void Send_CMD(void) {
  unsigned int i;
  for(i=0; i<4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}
unsigned char Checksum_cal(void) {
  unsigned char count, SUM=0;
  for(count=0; count<7; count++) {
     SUM += Receive_Buff[count];
  }
  return 256-SUM;
}
 
void setup() {
  pinMode(13,INPUT);
  pinMode(11,OUTPUT);
  Serial.begin(9600);
  while (!Serial) ;
  mySerial.begin(9600);
  while (!mySerial);
}
 
void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while(1) {
    if(mySerial.available()) { 
       Receive_Buff[recv_cnt++] = mySerial.read();
      if(recv_cnt ==8){recv_cnt = 0; break;}
    }
  } 
  
  if(Checksum_cal() == Receive_Buff[7]) {
     PPM_Value = Receive_Buff[3]<<8 | Receive_Buff[4];
     Serial.write("   PPM : ");
     Serial.println(PPM_Value);
  }
   else {
    Serial.write("CHECKSUM Error");
  }
  delay(1000);
} 


## Code execute2
'''
int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000;  // 30초 동안 샘플링
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float concentration = 0;

void setup()
{
    Serial.begin(9600);
    pinMode(pin, INPUT);
    starttime = millis();
    Serial.println("미세먼지 측정을 시작합니다...");
    Serial.println("==============================");
}

void loop()
{
    duration = pulseIn(pin, LOW);
    lowpulseoccupancy = lowpulseoccupancy + duration;

    if ((millis()-starttime) > sampletime_ms)  // 30초마다 측정
    {
        ratio = lowpulseoccupancy/(sampletime_ms*10.0);
        concentration = 1.1*pow(ratio,3)-3.8*pow(ratio,2)+520*ratio+0.62; // ug/m3 단위

        Serial.println("==============================");
        Serial.print("미세먼지 농도: ");
        Serial.print(concentration);
        Serial.println(" ug/m3");

        // 대기질 상태 표시
        Serial.print("대기질 상태: ");
        if(concentration <= 30) {
            Serial.println("좋음");
        }
        else if(concentration <= 80) {
            Serial.println("보통");
        }
        else if(concentration <= 150) {
            Serial.println("나쁨");
        }
        else {
            Serial.println("매우 나쁨");
        }

        Serial.println("------------------------------");
        Serial.print("측정 시간: ");
        Serial.print(millis()/1000);
        Serial.println("초");
        Serial.println("==============================\n");

        // 다음 측정을 위한 초기화
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
'''
## Execution Results

https://github.com/user-attachments/assets/70a55cd8-dd0c-42a2-abb7-f2050644d590



