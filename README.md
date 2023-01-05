# [IoT_project] Parking-System

주제 : 주차정보 안내 시스템

# 1. 시스템 개요 및 필요성
현재 지도 어플리케이션으로 주차장의 위치는 파악할 수 있다. 하지만 그 주차장의 잔여석을 파악할 수 있는 기능은 대부분 지원하지 않는다. 
이로 인해 도심지나 관광지의 경우 주차자리가 없어 불법 주차를 하는 것을 많이 볼 수 있다. 또한 주차로 인한 갈등 유발이 최근 들어 많아진 것을 뉴스를 통해 볼 수도 있었다. 
이러한 점에서 ‘주차장을 방문하기 전 미리 주차장의 상황(주차장 내부의 잔여석 여부)을 확인할 수 있는 방법은 없을까?’ 라는 생각을 통해 이 프로젝트를 진행하게 되었다.

(출처 : https://n.news.naver.com/mnews/article/277/0005133285?sid=102, https://n.news.naver.com/mnews/article/028/0002580269?sid=102)

# 2. 계획대비 변경내용
초음파 센서를 이용하여 주차장 잔여 공간 파악 데이터베이스 저장, 입구에 주차정보 LCD로 표시, FLask를 이용하여 웹으로 알려주며, 주차장 내에 CCTV도 웹으로 볼 수 있었던 내용은 거의 동일하나, 잔여공간 데이터베이스 저장은 제외하였고, 입구에서 차단기(Servo모터 대신 LED사용)를 이용하여 차량 정지시 초음파센서로 차량확인 후 카메라로 사진을 찍어 외부 컴퓨터로 사진을 전달하고, opencv코드를 이용하여 번호판값을 추출하였다. 그리고 그 값과
날짜값을 주차장 내부 라즈베리파이에 전송하여 데이터베이스에 저장하여 기록할 수 있게 하였다. 초음파 센서 개수가 제한적이기 때문에 주차장의 자리를 2자리로만 구현하였다. 

# 3. 최종 시스템 구성 및 기능

### 주차장 입구
1. 차량 입구 진입시 차단기 앞에 정지
2. 초음파 센서로 감지후 사진 촬영
3. 사진을 외부 컴퓨터로 전송하여 opencv로 번호판 값 추출
4. 번호판 값을 내부 라즈베리파이와의 공유폴더에 저장
5. flask를 이용하여 html 사용

---------------------------------------------------

### 주차장 내부
1. 주차장 내부의 CCTV역할을 하는 카메라 – motion을 감지하여 사진을 저장
2. 주차자리마다 초음파 센서와 LED(RED, GREEN)을 설치해 초음파 센서로 거리 감지 후
 주차 자리 가능 여부 LED로 알림
3. 감지한 값들을 boolean값으로 저장하여 LCD에 주차가능자리와 주차된 자리의 수를 파악하여
 표시 및 값들을 주차장 입구의 라즈베리파이로 전송한다.
- Flask
1. 주차장 내부에서 받은 각각의 주차자리 값으로 주차가능한 자리 표시
2. 주차장 내부의 라즈베리파이의 CCTV인 카메라를 읽어서 html에서 볼 수 있도록 함


# 4. 기능별 동작화면
1. 입구
LCD로 주차장에 들어가기 전 빈공간의 수와 주차된 공간의 수를 볼 수 있다. 2. 주차
각각 오른쪽 주차시, 왼쪽 주차시, 빈공간일 시 상황이며 빈공간일 때는
GREEN LED가 주차한 공간은 RED LED로 알려준다. 그때마다 LCD화면이 변경된다.
3. CCTV
motion 감지로 사진을 저장한다.
4. FLASK, HTML
HTML에서 CCTV로 주차장 상황을 알 수 있고, 주차 공간도 파악할 수 있다.
5. DATA BASE
클라우드 역할을 하는 PC에서 공유폴더에 txt파일을 저장해 그 값을 읽어 데이터 베이스에 저장한다.
6. 입구
차량 진입시 카메라 찍은 후 GREEN -> 차량통과 -> RED
7. OPENCV 과정


# 5. 문제점 및 해결방법
#### 1. 카메라 motion 문제
- 해결방안 : 기존의 UV4L 패키지와 충돌로 UV4L 패키지 삭제 후 해결

#### 2. Servo모터 전력문제
- 해결방안 : Servo모터 미작동으로 서치한 결과 라즈베리파이에서 제공하는 전력에서는 Servo모터
작동이 힘들다는 내용을 찾았고, 이를 대신에 LED를 적용하여
차단기가 닫혀있을 때는 RED
차단기가 열릴 때는 GREEN

#### 3-1. socket 통신 문제 (bad file descriptor)
- 해결방안 : server while문 내에서 코드 첫 부분에 연결하고 마지막에 부분에서 연결을 해제하여 새로 연결하고 client에서 연결하여 전송, 즉 server에서는 서버를 계속하여 reset시키고 client에서는 특별한 경우일 때마다 연결하여 전송하고 연결을 종료하도록 하였다. 

#### 3-2. [연장 문제] socket flask충돌 문제
같은 코드내에서 충돌하는 문제가 발생하여 코드를 분리하고, flask를 background로 실행해 해결하였다.

#### 4. 카메라 화면 상하 반전 문제
- 4-1 cctv 해결방안 : motion.conf nano편집기를 사용하여 rotate 180으로 반전시켜 촬영하였다. 
- 4-2 입구 번호판 촬영 해결방안 : 찍고 openCv 코드내에서 사진을 뒤집어 인식하였다. 

#### 5. OpenCV 라즈베리파이 사용시 성능부족
-해결방안 : 라즈베리파이에서 클라우드역할을 하는 다른 PC와 공유하는 폴더에 저장하고, PC는 코드를 반복하여 실행하고, 반복할때마다 저장된 폴더의 jpg파일 개수를 비교하여 새로운 사진이 저장될 시 그 사진에서 번호판 값을 추출해 또 다른 라즈베리파이와 공유하는 폴더에 텍스트파일로 추가하여 저장한다.

------------------------------------


# 6. 최종 구성요소

#### 1. 초음파 센서
 - 주차자리에서 거리 측정으로 주차 확인
 - 입구에서 차량확인

#### 2. LCD
 - 주차 자리 알림으로 주차장 내부에 빈 공간과 주차된 공간 여부 수치화시켜 I2C화면에 출력
 
#### 3. pi camera
 - 주차장 내부 CCTV(motion)으로 감지
 - 주차장 입구에서 초음파센서 2역할로 차량확인 후 카메라를 통해 번호판 촬영

#### 4. led
 - 주차자리의 주차 가능 여부알림
 - servo motor의 전력문제로 led로 대신하여 red = 차단기로 입구 차단 / green = 차단기 open


# 7. 개인별 역할
### 정승조 (팀장)
- 초음파 센서를 이용해 차량이 들어온 것을 확인하고, 파이 카메라를 통해 차량의 번호판을 촬영합니다.
- 이를 클라우드 역할을 하는 PC로 넘겨주고 그 PC는 OpenCV를 통해 차량 번호판을 인식하여 인식한 번호를 상대 라즈베리파이의 텍스트파일에 접근하여 작성합니다.
- 또한 Flask를 활용하여 웹 서버를 통해 사용자들에게 미리 주차장의 현황을 알 수 있도록 합니다. 
- 사용자는 웹 어플리케이션을 통해 주차장에 남은 좌석을 확인할 수 있고, 상대방의 라즈베리파이의 카메라를 CCTV로 활용하여 실시간으로 주차장의 상태를 파악할 수 있도록 구현하였습니다.

### 이정우
- motion을 이용하여 카메라로 스트리밍하고 motion감지시 사진을 그때마다 사진을 저장할 수 있도록 하였다. 
- 초음파 센서를 통해 주차자리의 주차여부를 파악해 led를 통해 나타내었고, 주차공간의 값들을 LCD화면에 출력하였다. 또한 다른 라즈베리파이에 자리 각각의 주차여부의 값(boolean -> str)형태로 변환하여 socket통신으로 전달해 주었다. 
- 정승조 팀장이 공유하는 폴더에 입구에서 인식한 번호판과 시간을 입력한텍스트 파일을 읽어 mysql에 기록하였다. 


---------------------------------------------
# 8. 프로젝트 구현 과정

#### - Pi 카메라로 차량을 인식하고 촬영한 후 OpenCV를 통해 차량 번호를 인식하는 과정

![image](https://user-images.githubusercontent.com/84575041/210685588-6bf0e412-3795-4727-8488-369a2700347d.png)


![image](https://user-images.githubusercontent.com/84575041/210685601-3105f724-6d48-4b76-9128-5929eff38bbc.png)

![image](https://user-images.githubusercontent.com/84575041/210685742-463688b0-ba43-4952-8c7c-fe702d8574f9.png)

![image](https://user-images.githubusercontent.com/84575041/210685747-7f08ae7d-a881-4f40-a945-f7ed2716496b.png)

![image](https://user-images.githubusercontent.com/84575041/210685751-8a6d9e23-c6ac-4407-a176-6b36b75ee089.png)

--------------------------------------------------------------------------------------

#### Flask (Python)을 통해 웹페이지 구현
![image](https://user-images.githubusercontent.com/84575041/210685829-7ba53eaf-c2c0-4599-98b2-d61d6293b7d6.png)

![image](https://user-images.githubusercontent.com/84575041/210685833-162ba5ff-c941-4458-9e65-4ddca3ee3f96.png)

