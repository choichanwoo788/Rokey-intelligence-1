# Rokey Intelligence 1

건설 현장 중장비 주변의 안전 관리를 보조하기 위한 SLAM 기반 자율주행 로봇 시스템입니다. 2대의 AMR이 리더-팔로워 구조로 협력하고, YOLO 객체 인식과 Nav2 자율주행을 결합해 작업자, 차량, 장애물 상황을 감지합니다.

## 프로젝트 목표

- 중장비 이동 중 발생할 수 있는 충돌 위험을 줄입니다.
- 인력 중심 안전 감시의 사각지대와 운영 부담을 보완합니다.
- AMR 2대의 협력 주행으로 현장 주변을 모니터링합니다.
- SLAM, Nav2, YOLO 기반 인식으로 이동과 위험 감지를 자동화합니다.

## 주요 기능

- 리더 AMR Nav2 이동 제어
- 팔로워 AMR 추종 주행
- OAK-D / 웹캠 기반 이미지 수신
- YOLO 기반 작업자, 차량, 지게차, 장애물 인식
- RGB-D 기반 객체 거리 추정
- Flask 기반 모니터링 화면
- ROS2-웹 브리지 서버

## 기술 스택

- Ubuntu 22.04
- ROS2 Humble
- Python
- Flask
- OpenCV
- cv_bridge
- ultralytics YOLO
- Nav2
- TurtleBot4 / AMR
- OAK-D, WebCam, LiDAR

## 폴더 구조

```text
src/
  AMR_leader_Nav2.py          # 리더 AMR Nav2 주행 노드
  AMR_follower_Nav2.py        # 팔로워 AMR Nav2 주행 노드
  AMR_follower_tracking.py    # 팔로워 추종 제어 노드
  AMR_follower_yolo_depth.py  # 팔로워 RGB-D YOLO 거리 추정 노드
  AMR_leader_yolo_detect.py   # 리더 카메라 YOLO 위험 감지 노드
  YOLO_start_point.py         # 출발 지점 웹캠 객체 감지
  YOLO_A_zone.py              # A 구역 객체 감지
  YOLO_B_zone.py              # B 구역 객체 감지
  truck_fork_classifer_01.py  # truck/fork 감지 노드
  bridge_node.py              # ROS2 데이터를 Flask API로 연결
  app.py                      # Flask 모니터링 웹 앱
```

## 시스템 실행 방법

### 1. 기본 환경 준비

ROS2 Humble 환경을 활성화하고 필요한 Python 패키지를 설치합니다.

```bash
source /opt/ros/humble/setup.bash
pip install flask opencv-python ultralytics numpy
```

ROS2 Python 패키지는 실행 환경에 따라 `rclpy`, `sensor_msgs`, `geometry_msgs`, `nav2_msgs`, `cv_bridge`, `tf2_ros` 등이 필요합니다.

### 2. YOLO 모델 경로 확인

일부 파일은 코드 내부에 절대 경로로 모델을 참조합니다.

```text
/home/rokey/rokey_ws/src/ch_final_project/amr_yolon_26424.pt
/home/rokey/rokey_ws/src/yolo_web/yolo_web/wc_last.pt
```

실행 전 로컬 모델 위치에 맞게 `model_path` 값을 수정하거나 동일 경로에 모델 파일을 배치해야 합니다.

### 3. ROS2 노드 실행

각 터미널에서 필요한 노드를 개별 실행합니다.

```bash
cd src
python3 AMR_leader_Nav2.py
python3 AMR_follower_Nav2.py
python3 AMR_follower_tracking.py
python3 AMR_leader_yolo_detect.py
python3 AMR_follower_yolo_depth.py
```

웹캠 기반 구역 감지는 필요 구역에 맞게 실행합니다.

```bash
python3 YOLO_start_point.py
python3 YOLO_A_zone.py
python3 YOLO_B_zone.py
python3 truck_fork_classifer_01.py
```

### 4. 웹 브리지와 모니터링 서버 실행

`bridge_node.py`는 ROS2 구독 데이터와 Flask API를 함께 실행하며, 기본 포트는 `5001`입니다.

```bash
python3 bridge_node.py
```

`app.py`는 모니터링 웹 앱이며, 기본 포트는 `5000`입니다.

```bash
python3 app.py
```

브라우저에서 접속합니다.

```text
http://localhost:5000
```

## 실행 전 확인 사항

- 카메라 토픽 이름이 코드와 실제 장치 설정과 일치해야 합니다.
- Nav2, map, localization, TF 구성이 먼저 정상 동작해야 합니다.
- CUDA가 없는 환경에서는 `.to('cuda')` 사용 부분을 CPU 실행에 맞게 수정해야 합니다.
- 웹캠 인덱스와 OAK-D compressed image 토픽을 실제 환경에 맞게 확인해야 합니다.
