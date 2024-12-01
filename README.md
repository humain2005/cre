주제: 온습도 알리미
조: 4팀
팀원: 최태인, 정숭훈, 권예슬, 강민수

개발 목표: 이 프로그램은 아두이노 알리미를 활용하여 주변 환경의 온도와 습도를 실시간으로 측정한 뒤, 이를 바탕으로 사용자가 쾌적한 환경을 조성할 수 있도록 돕는 시스템으로,. 온도와 습도에 따라 LED와 피에조 스피커, 모터 등을 제어하여, 환경 변화에 대한 실시간 피드백을 제공할 수 있도록 설계되었다. 이를 통해 사용자는 쾌적한 온도와 습도 범위 내에서 효율적으로 환경을 관리할 수 있다.

사용 부품: ABot, LED, 점퍼선, 피에조 스피커, 블루투스 모듈, 저항(220옴), 온습도 센서, 서브 모터

개발 과정
11월 13일 팀프로젝트에서 여러 차례의 논의 끝에 주제로 ‘스마트 가습기 제어 시스템’ 선정
11월 19일 온습도 센서 준비, 기초적인 회로도 구성, 온습도 센서로부터 값을 받아오는 기초적인 형태의 아두이노 코드 작성, 과제 진행 보고서 제출
11월 23일 개발 목표 및 필요 부품 확립, LED 장착(적정 범위보다 높은지 낮은지에 따라 각기 다른 LED 작동), 피에조 스피커(범위에서 벗어날 시 작동) 장착 계획서 작성
11월 26일부터30일 몇 차례의 논의 끝에 가습기 모듈 장착에 대한 시간적, 기술적 문제 등으로 가습기 모듈 장착을 포기하고, 그 대신 온도 측정 기능 추가, 앱인벤터에서 수치에 따른 색 출력 기능 추가, 온도와 습도 최고 기록 추가, 범위에서 벗어난 만큼 피에조 스피커 음량 증가 기능 추가, 수치에 따른 Abot 움직임 추가
12월 2일부터 3일 긴급 회의 모집, 영상 촬영 및 편집, 포스터 제작, 검토, 결과물 제출
12월 4일 발표

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

      // Abot이 제자리에서 춤추기
      dance();
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
}

void rotateLeft() {
  leftWheel.write(60);   // 왼쪽 바퀴 후진
  rightWheel.write(120); // 오른쪽 바퀴 전진
  delay(500);            // 회전 0.5초
}

void dance() {
  rotateRight();
  delay(250);            // 잠시 멈춤
  rotateLeft();
  delay(250);            // 잠시 멈춤
}

해설
사용된 라이브러리
1.Servo: 서보 모터를 제어하기 위해 사용합니다. 서보 모터는 Abot의 바퀴 역할을 하며, 이동과 회전을 담당합니다.
-attach(pin): 지정된 핀에 서보 모터 연결.
-write(angle): 서보 모터의 각도를 설정하여 정지 또는 이동 동작 수행.
2.DHT: DHT11 센서에서 온도와 습도를 읽어옵니다.
-readHumidity(): 현재 습도를 반환.
-readTemperature(): 현재 온도를 반환.

DHT11 센서
1.역할: 주변 환경의 온도와 습도를 측정합니다.
2.코드 동작:
-dht.readHumidity(): 현재 습도를 읽어와 변수 h에 저장.
-dht.readTemperature(): 현재 온도를 읽어와 변수 t에 저장.
-오류 처리: 값이 유효하지 않을 경우 메시지를 출력하고 return으로 중단.
if (isnan(h) || isnan(t)) {
    Serial.println("센서 읽기 실패!");
    return;
}

LED
1.역할: 온도와 습도 상태에 따라 Abot의 환경 상태를 시각적으로 표시합니다.
2.LED 핀 정의:
-LEDPIN_LOW (5번 핀): 경계 상태를 나타냄.
-LEDPIN_MID (6번 핀): 쾌적한 상태를 나타냄.
-LEDPIN_HIGH (3번 핀): 위험 상태를 나타냄.
LED 제어 조건:
1.쾌적한 상태:
-습도: 40~60%
-온도: 20~26°C
-동작: LEDPIN_MID 켜짐
if (h >= 40 && h <= 60 && t >= 20 && t <= 26) {
    digitalWrite(LEDPIN_MID, HIGH);
}
2.경계 상태:
-습도: 3039% 또는 6170%
-온도: 1519°C 또는 2630°C
-동작: LEDPIN_LOW 켜짐.
if ((h >= 30 && h <= 39) || (h >= 61 && h <= 70) || 
    (t >= 15 && t <= 19) || (t >= 26 && t <= 30)) {
    digitalWrite(LEDPIN_LOW, HIGH);
}
3.위험 상태:
-습도: 30% 미만 또는 70% 초과
-온도: 15°C 미만 또는 30°C 초과
-동작: LEDPIN_HIGH 켜짐, 스피커 경고음 발생, Abot이 회전.
if ((h < 30 || h > 70) || (t < 15 || t > 30)) {
    digitalWrite(LEDPIN_HIGH, HIGH);
    tone(BUZZERPIN, buzzerFreq, 500);
    dance();
}

서보 모터
역할: Abot의 바퀴를 제어하여 이동과 회전 동작을 수행합니다.
핀 정의:
-왼쪽 모터: 12번 핀.
-오른쪽 모터: 13번 핀.
제어 방법:
1.정지:
leftWheel.write(90);
rightWheel.write(90);
2.회전:
leftWheel.write(120); // 왼쪽 바퀴 전진
rightWheel.write(60);  // 오른쪽 바퀴 후진
delay(500);            // 0.5초 동안 회전

피에조 스피커
역할: 온도와 습도가 정상 범위를 벗어난 경우 경고음을 발생시킵니다.
특징:
-온도와 습도의 이상 정도에 따라 소리의 주파수를 변경합니다.
int h_diff = abs(h - 50);  // 목표 습도 50%에서 차이
int t_diff = abs(t - 23);  // 목표 온도 23도에서 차이
buzzerFreq = 1000 + (h_diff + t_diff) * 10; // 차이에 비례하여 주파수 증가
tone(BUZZERPIN, buzzerFreq, 500);

초기화 (setup())
역할:
-DHT 센서와 서보 모터를 초기화.
-LED와 스피커 핀을 출력으로 설정.
-서보 모터는 초기 상태로 정지 설정.
leftWheel.write(90);  // 정지
rightWheel.write(90); // 정지
delay(1000);          // 초기화 대기

환경 측정 및 반응 (loop())
-1.25초 간격으로 센서 값 읽기
if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
}
-온도와 습도 측정:
dht.readHumidity()와 dht.readTemperature()로 값을 읽어옴.
-LED, 스피커, 모터 제어:
측정된 값에 따라 LED 상태와 스피커 주파수 설정.
경계 또는 위험 상태일 경우, Abot이 회전 또는 정지.

환경 상태별 동작 요약
1.쾌적한 상태:
-MID LED 점등.
-Abot 정지.
digitalWrite(LEDPIN_MID, HIGH);
leftWheel.write(90);
rightWheel.write(90);
2.경계 상태:
-LOW LED 점등.
-Abot 정지.
digitalWrite(LEDPIN_LOW, HIGH);
leftWheel.write(90);
rightWheel.write(90);
3.위험 상태:
-HIGH LED 점등.
-스피커 경고음.
-Abot 회전(춤).
digitalWrite(LEDPIN_HIGH, HIGH);
tone(BUZZERPIN, buzzerFreq, 500);
dance();
시리얼 모니터 출력
현재 상태:
1.온도: t
2.습도: h
3.최고 온도와 습도 기록: tt와 hh.
Serial.println("h" + String(h));
Serial.println("t" + String(t));
Serial.println("a" + String(hh));
Serial.println("b" + String(tt));

앱인벤터
시리얼 모니터에서 출력된 값들을 읽어온 뒤, 문자열 제일 앞의 한 글자를 인식한 뒤, 해당 값이 앱
인벤터 내의 전역변수 imsi와 비교 후, 그 뒤의 숫자를 전역변수 value에 할당한 다음, value의 값
을 특정 레이블에 표기한다. 이때 value가 특정 범위보다 높거나 낮을 경우, 화면에 붉은색 혹은 푸
른색이 나타나도록 하며, 특정 범위 내에 있을 경우 흰색만을 나타나게 한다. 

각 조원 역할
최태인: 의견 제시, 검토, 계획서 작성, 회로 설계, 부품 준비, 조립, 회의록 작성, 아두이노 코드 작성, 앱인벤터 작성, 오류 수정, 영상 촬영, ppt 작성, 포스터 제작, 대본 작성, 보고서 작성
정숭훈: 의견 제시, 검토, 주제 제시, 기능 구상, 부품 준비, 조립, 영상 촬영, 영상 편집, 포스터 제작, 유튜브 업로드
권예슬: 의견 제시, 검토, 주제 제시, 기능 구상, 부품 준비, 대본 작성, 더빙 녹음
강민수: 의견 제시, 검토, 대본 작성, 포스터 제작

소감
최태인

정숭훈

권예슬

강민수


