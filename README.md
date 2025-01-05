# Jetson_Nano_Project

## Supplies for a project

### 1. JetsonNano
![KakaoTalk_20241219_195505659_01](https://github.com/user-attachments/assets/f330eeec-3e65-4dcf-930a-e8337a67c4da)
### 2. Arduino, Sensor : CM1106, Grove dust sensor
![KakaoTalk_20241219_195505659](https://github.com/user-attachments/assets/74ef87ed-2d30-49b3-8ba1-9fe869cce487)
### 3. Arduino-cm1106 connection circuit diagram
![스크린샷 2025-01-05 235814](https://github.com/user-attachments/assets/df1ce120-caa3-4e01-b64a-8f89d7f33d0d)
### 4. Arduino-dust sensor connection circuit diagram
![image](https://github.com/user-attachments/assets/cde6fb73-8839-435e-b36a-4925e2417f8f)

## Install Arduino

### 1. Download version 1.8.19 from the Home Page : https://www.arduino.cc/en/software
* Caution : You can't run dust sensor unless you download version 1.18.19
### 2. Unzip Arduino files : 

cd Downloads
tar -xf arduino-1.8.19-linuxaarch64.tar.xz 
cd arduino-1.8.19

### 3. Arduino Installation/Setup : 

sudo ./install.sh
sudo usermod -aG dialout $USER
newgrp dialout

arduino (→ Practice)

## Arduino code execute

### 1. Enter the code below 

### 2. Press the button ![image](https://github.com/user-attachments/assets/0c1998a9-db98-4fa6-8f3a-ed514e293162)

### 3. Press the button ![image](https://github.com/user-attachments/assets/e8a8095e-f878-45d6-8650-89d3edad0856)



##


## CO2 Sensor execute code 
```
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
```

## Dust sensor execute code 
```
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
```
## Execution Results Videos

https://github.com/user-attachments/assets/70a55cd8-dd0c-42a2-abb7-f2050644d590

## Reference : Create a chatbot using function calling

```
import gradio as gr
import random
import os
from openai import OpenAI
import pandas as pd
from urllib.request import urlopen

# OpenAI Key 설정
os.environ['OPENAI_API_KEY'] = 
OpenAI.api_key = os.getenv("OPENAI_API_KEY")

# API 데이터 가져오기
url = 'https://apihub.kma.go.kr/api/typ01/url/kma_pm10.php?tm1=202412112020&tm2=202412122020&authKey=8diTRQm4TEKYk0UJuOxCsg'
with urlopen(url) as f:
    html = f.read().decode('euc-kr')

lines = html.split('\n')
filtered_lines = [line for line in lines if ',   132,' in line]

columns = ["TM", "STN", "PM10", "FLAG", " ", "MQC"]
data = []
for line in filtered_lines:
    values = line.split(',')
    data.append([value.strip() for value in values])

df = pd.DataFrame(data, columns=columns[:len(data[0])])
df.drop(['STN', 'FLAG', ' ', 'MQC'], axis=1, inplace=True)
last_API_value = float(df['PM10'].iloc[-1])

# 젯슨 나노 센서 데이터 가져오기
df_dust = pd.read_csv('dust_sensor_data.csv')
last_sensor_value = float(df_dust['Dust Concentration (ug/m3)'].iloc[-1])

# OpenAI function 정의
def get_last_sensor_value():
    """젯슨 나노 센서의 최신 값 반환"""
    return {"last_sensor_value": last_sensor_value}

def get_last_api_value():
    """API의 최신 PM10 값 반환"""
    return {"last_api_value": last_API_value}

functions = [
    {
        "name": "get_last_sensor_value",
        "description": "젯슨 나노 센서의 최근 먼지 농도 값을 반환",
        "parameters": {
            "type": "object",
            "properties": {},
            "required": []
        }
    },
    {
        "name": "get_last_api_value",
        "description": "API의 최근 PM10 값을 반환",
        "parameters": {
            "type": "object",
            "properties": {},
            "required": []
        }
    }
]

def process(user_message, chat_history):
    # 특정 질문에 대한 직접 처리 로직
    if "대전 날씨" in user_message:
        if last_sensor_value > last_API_value:
            ai_message = (f"현재 젯슨 나노 측정 값({last_sensor_value} ug/m3)가 "
                          f"API PM10 값({last_API_value} ug/m3)보다 높습니다. 환기를 시키세요.")
        else:
            ai_message = (f"현재 젯슨 나노 측정 값({last_sensor_value} ug/m3)가 "
                          f"API PM10 값({last_API_value} ug/m3)보다 낮거나 같습니다. 공기청정기를 사용하세요.")
        chat_history.append((user_message, ai_message))
        return "", chat_history

    # 위 조건에 해당하지 않으면 OpenAI function calling 사용
    client = OpenAI()
    completion = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": user_message}
        ],
        functions=functions,
        function_call="auto"
    )

    response = completion.choices[0].message

    # function calling 응답 처리
    if "function_call" in response:
        func_name = response["function_call"]["name"]
        if func_name == "get_last_sensor_value":
            func_result = get_last_sensor_value()
        elif func_name == "get_last_api_value":
            func_result = get_last_api_value()
        else:
            func_result = {}

        # function 호출 결과를 다시 모델에 전달하여 최종 답변 생성
        completion = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": user_message},
                {"role": "function", "name": func_name, "content": str(func_result)}
            ]
        )
        ai_message = completion.choices[0].message.content
    else:
        # 일반 답변
        ai_message = response["content"]

    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="채팅창")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

## Function Calling execute video
https://github.com/user-attachments/assets/564c2f67-d021-4a3e-b1c4-44a54eac2719







