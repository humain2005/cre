주제: 온습도 알리미
조: 4팀
팀원: 최태인, 정숭훈, 권예슬, 강민수

개발 목표: 이 프로그램은 아두이노 알리미를 활용하여 주변 환경의 온도와 습도를 실시간으로 측정한 뒤, 이를 바탕으로 사용자가 쾌적한 환경을 조성할 수 있도록 돕는 시스템으로,. 온도와 습도에 따라 LED와 피에조 스피커, 모터 등을 제어하여, 환경 변화에 대한 실시간 피드백을 제공할 수 있도록 설계되었다. 이를 통해 사용자는 쾌적한 온도와 습도 범위 내에서 효율적으로 환경을 관리할 수 있다.

사용 부품: ABot, LED, 점퍼선, 피에조 스피커, 블루투스 모듈, 저항(220옴), 온습도 센서, 서브 모터

아두이노 코드:
#include <Servo.h> // Servo 라이브러리 추가
#include <DHT.h>

#define DHTPIN 2     // DHT 센서가 연결된 핀 번호
#define DHTTYPE DHT11   // DHT 11 센서 사용
#define LEDPIN_LOW 5    // 습도 30~39% 또는 61~70%, 온도 15~19 또는 26~30일 때 LED (5번 핀)
#define LEDPIN_MID 6    // 습도 40~60%이고 온도 20~26일 때 LED (6번 핀)
#define LEDPIN_HIGH 3   // 습도 30% 미만 또는 70% 초과, 온도 15도 미만 또는 30도 초과일 때 LED (3번 핀)
#define BUZZERPIN 4     // 피에조 스피커를 4번 핀에 연결

DHT dht(DHTPIN, DHTTYPE);
Servo leftWheel;  // 왼쪽 바퀴 서보 모터
Servo rightWheel; // 오른쪽 바퀴 서보 모터

unsigned long previousMillis = 0;
const long interval = 1250; // 1.25초 간격

float hh = 0; // 최고 습도 기록
float tt = 0; // 최고 온도 기록

void setup() {
  Serial.begin(9600);
  dht.begin();
  pinMode(LEDPIN_LOW, OUTPUT);
  pinMode(LEDPIN_MID, OUTPUT);
  pinMode(LEDPIN_HIGH, OUTPUT);
  pinMode(BUZZERPIN, OUTPUT);

  leftWheel.attach(12);  // 왼쪽 바퀴 서보 모터 연결 (핀 12)
  rightWheel.attach(13); // 오른쪽 바퀴 서보 모터 연결 (핀 13)

  // 서보 초기화
  leftWheel.write(90);  // 정지
  rightWheel.write(90); // 정지
  delay(1000);          // 초기화 대기
}

void loop() {
  unsigned long currentMillis = millis();

  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;

    // 센서 읽기 코드
    float h = dht.readHumidity();
    float t = dht.readTemperature();

    if (isnan(h) || isnan(t)) {
      // 센서 읽기 실패 시 경고만 출력하고 동작 유지
      Serial.println("센서 읽기 실패!");
      return;
    }

    // 결과 출력
    Serial.println("h" + String(h));
    Serial.println("t" + String(t));

    // 최고 기록 업데이트
    if (h > hh) hh = h;
    if (t > tt) tt = t;
    Serial.println("a" + String(hh));
    Serial.println("b" + String(tt));

    // 모든 LED 끄기
    digitalWrite(LEDPIN_LOW, LOW);
    digitalWrite(LEDPIN_MID, LOW);
    digitalWrite(LEDPIN_HIGH, LOW);

    // 모든 모터 정지
    leftWheel.write(90);  // 정지
    rightWheel.write(90); // 정지

    // 습도와 온도에 따른 LED 제어
    int buzzerFreq = 1000; // 기본 소리 주파수 (1000Hz)

    if ((h < 30 || h > 70) || (t < 15 || t > 30)) {
      digitalWrite(LEDPIN_HIGH, HIGH);

      // 범위를 벗어난 정도에 비례하여 소리 크기 증가
      int h_diff = abs(h - 50);  // 목표 습도 50%에서 차이
      int t_diff = abs(t - 23);  // 목표 온도 23도에서 차이
      buzzerFreq = 1000 + (h_diff + t_diff) * 10; // 차이에 비례하여 소리 주파수 증가

      tone(BUZZERPIN, buzzerFreq, 500);  // 계산된 주파수로 소리 낸다

      // Abot이 오른쪽으로 회전 (지속적으로 빙글빙글)
      rotateRight();
    } else if ((h >= 30 && h <= 39) || (h >= 61 && h <= 70) || 
               (t >= 15 && t <= 19) || (t >= 26 && t <= 30)) {
      digitalWrite(LEDPIN_LOW, HIGH);

      // Abot이 가만히 있음
      leftWheel.write(90);  // 정지
      rightWheel.write(90); // 정지
    } else if (h >= 40 && h <= 60 && t >= 20 && t <= 26) {
      digitalWrite(LEDPIN_MID, HIGH);

      // Abot 정지
      leftWheel.write(90);
      rightWheel.write(90);
    }
  }
}

void rotateRight() {
  leftWheel.write(120);  // 왼쪽 바퀴 전진
  rightWheel.write(60);  // 오른쪽 바퀴 후진
  delay(500);            // 회전 0.5초
  //실제론 끊임없이 값을 받아 움직임 값이 초기화되기 때문에 마치 춤추는 것 같은 모양새가 된다.
}
해설
사용된 라이브러리
-Servo: 서보 모터의 제어를 위한 라이브러리.
-DHT: DHT11 센서의 온도와 습도 데이터를 읽는 데 사용.

DHT11 센서
주변 환경의 온도와 습도를 측정.
readHumidity()와 readTemperature() 함수를 사용하여 값을 읽음.
측정값이 유효하지 않으면 오류 메시지(Serial.println("센서 읽기 실패!");)를 출력
if (h >= 40 && h <= 60 && t >= 20 && t <= 26) {
    digitalWrite(LEDPIN_MID, HIGH);
}

LED
세 개의 LED 핀(LOW: 5번, MID: 6번, HIGH: 3번)은 환경 상태를 나타냄.
예를 들어, 쾌적한 상태에서는 MID LED를 점등

서보 모터
두 개의 서보 모터(12번, 13번 핀)는 Abot의 바퀴를 제어.
정지
leftWheel.write(90);
rightWheel.write(90);
회전(다만 본 코드에서는 값 초기화까지 고려하여 실질적으로는 춤추는 코드로 쓰임)
leftWheel.write(120); // 왼쪽 바퀴 전진
rightWheel.write(60);  // 오른쪽 바퀴 후진

피에조 스피커
온도와 습도가 임계치를 벗어난 경우 경고음을 발생.
벗어난 정도에 따라 소리의 주파수가 증가
int h_diff = abs(h - 50);  // 목표 습도 50%와 차이
int t_diff = abs(t - 23);  // 목표 온도 23도와 차이
int buzzerFreq = 1000 + (h_diff + t_diff) * 10;
tone(BUZZERPIN, buzzerFreq, 500);

초기화 (setup())
DHT 센서와 서보 모터를 초기화.
LED와 스피커 핀의 출력 설정.
서보 모터는 정지 상태로 시작

환경 측정 및 반응 (loop())
매 1.25초마다 온도와 습도를 측정.
값에 따라 LED, 스피커, 모터를 제어

환경 상태별 동작:
쾌적한 상태 (습도 40~60%, 온도 20~26도):
MID LED 켜짐, Abot 정지.
digitalWrite(LEDPIN_MID, HIGH);

경계 상태 (습도 30~39% 또는 61~70%, 온도 15~19도 또는 26~30도):
LOW LED 켜짐, Abot 정지.
digitalWrite(LEDPIN_LOW, HIGH);

위험 상태 (습도 30% 미만 또는 70% 초과, 온도 15도 미만 또는 30도 초과):
HIGH LED 켜짐, 스피커 경고음 재생, Abot 회전.
digitalWrite(LEDPIN_HIGH, HIGH);
tone(BUZZERPIN, buzzerFreq, 500);
rotateRight();

모터 회전 (rotateRight())
Abot이 오른쪽으로 회전하는 동작:
leftWheel.write(120);
rightWheel.write(60);
delay(500);

시리얼 모니터 출력
현재 측정된 온도와 습도를 출력하고, 최고 기록도 함께 표시
Serial.println("h" + String(h));
Serial.println("t" + String(t));
Serial.println("a" + String(hh))
Serial.println("b" + String(tt));

앱인벤터
시리얼 모니터에서 출력된 값들을 읽어온 뒤, 문자열 제일 앞의 한 글자를 인식한 뒤, 해당 값이 앱
인벤터 내의 전역변수 imsi와 비교 후, 그 뒤의 숫자를 전역변수 value에 할당한 다음, value의 값
을 특정 레이블에 표기한다. 이때 value가 특정 범위보다 높거나 낮을 경우, 화면에 빨간색 혹은 푸
른색이 나타나도록 하며, 특정 범위 내에 있을 경우 흰색만을 나타나게 한다. 

각 조원 역할
최태인: 의견 제시, 검토, 계획서 작성, 회로 설계, 부품 준비 및 조립, 회의록 작성, 코드 작성, 앱인벤터 작성, 영상 촬영, ppt 작성, 보고서 작성
정숭훈: 의견 제시, 검토, 앱인벤터 작성 보조, 영상 촬영, 영상 편집, 포스터 제작
권예슬: 의견 제시, 검토, 부품 준비 및 조립, 영상 촬영, 대본 작성, 영상 더빙
강민수: 의견 제시, 검토, 대본 작성

소감
최태인

정숭훈

권예슬

강민수


