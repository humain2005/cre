주제: 온습도 알리미
조: 4팀
팀원: 최태인, 정숭훈, 권예슬, 강민수

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
