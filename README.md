# 🛡️ RADAR
### Risk Aware Detection And Recognition
**전쟁상황 기반 자율 정찰 로봇**

> TurtleBot3 Burger 기반의 자율주행 정찰 로봇으로, 실시간 SLAM과 YOLO 객체 탐지를 통합하여  
> 미지의 전장 환경에서 군인·전차를 탐지하고 위협 위치를 자동으로 지도에 마킹합니다.

> **공개 저장소 범위:** 팀 프로젝트 중 제가 담당한 Reactive 주행 알고리즘, 탐지 좌표 융합, YOLO 추론 및 모델 실험 결과를 중심으로 정리했습니다. Qt HUD와 장비별 실행 스크립트는 팀원의 개인 개발 환경에서 관리되어 이 저장소에는 포함되어 있지 않습니다.

<br>

<p align="center">
  <img src="assets/radar_logo.png" width="300"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/ROS2-Humble-blue?logo=ros"/>
  <img src="https://img.shields.io/badge/Python-3.10-yellow?logo=python"/>
  <img src="https://img.shields.io/badge/YOLO-v11-purple"/>
  <img src="https://img.shields.io/badge/Platform-TurtleBot3-green"/>
  <img src="https://img.shields.io/badge/Period-2026.04.13~04.27-gray"/>
</p>

---

## 📋 목차

- [프로젝트 배경](#-프로젝트-배경)
- [시연 영상](#-시연-영상)
- [팀 구성](#-팀-구성)
- [시스템 구조](#-시스템-구조)
- [주요 기능](#-주요-기능)
- [AI · Perception](#-ai--perception)
- [Path Algorithm](#-path-algorithm)
- [Qt 전술 HUD](#-qt-전술-hud)
- [성능 지표](#-성능-지표)
- [기술 스택](#-기술-스택)
- [개발 환경 및 시스템 실행 흐름](#-개발-환경-및-시스템-실행-흐름)
- [트러블 슈팅](#-트러블-슈팅)
- [향후 계획](#-향후-계획)

---

## 🎯 프로젝트 배경
위험 환경을 사람이 직접 정찰하면 안전 문제가 발생하고, 원격 조작만으로는 미지 공간을 지속적으로 탐색하기 어렵습니다. RADAR는 TurtleBot3에 SLAM, 객체 탐지, 좌표 변환, Reactive 주행을 결합해 다음 흐름을 자동화한 팀 프로젝트입니다.

1. LiDAR로 미지 공간의 지도를 생성합니다.
2. 카메라 영상에서 위협 객체를 탐지합니다.
3. 탐지 결과를 지도 좌표로 변환해 위치를 표시합니다.
4. 위협 구역을 회피하며 미탐색 구역을 우선 순찰합니다.

---

## 🎬 시연 영상

| # | 내용 | 링크 |
|---|------|------|
| 1 | 실시간 자율주행 (1) | [▶ YouTube](https://youtube.com/shorts/bUdc0pBeM_Q) |
| 2 | 실시간 자율주행 (2) | [▶ YouTube](https://youtube.com/shorts/fysW5ICRKj0) |


| Qt HUD 실행 화면 | 실제 주행 환경 | SLAM 맵 생성 결과 |
|:---:|:---:|:---:|
| <img src="https://github.com/user-attachments/assets/602f4f2d-4024-4984-87dd-25ba4aba7e11" width="320"/> | <img src="assets/demo_field.jpg" width="320"/> | <img src="assets/slam_map.png" width="320"/> |

---

## 👥 팀 구성

**소속:** Intel 9기  
**프로젝트 기간:** 2026.04.13 ~ 2026.04.27

| 역할 | 이름 | 담당 |
|:---:|:---:|:---|
| **TL** | 윤성진 | SLAM · 실시간 매핑 및 자율주행 |
| **DTL / PM** | 배현규 | AI Perception · YOLO 학습 · Depth Estimation ML |
| **LE** | 안형준 | Overall Development · 시스템 통합 · Reactive 알고리즘 |
| **AE** | 박상호 | Simul UI/UX · Qt HUD · Gazebo 시뮬레이션 |

---

## 🏗️ 시스템 구조

<img src="assets/System Architecture.png" width="991" height="753"/>

### 시스템 실행 흐름

1. Raspberry Pi에서 TurtleBot3 bringup과 YOLO 추론 노드를 실행
2. Ubuntu PC에서 SLAM Toolbox를 실행하고 `map` 프레임 생성 확인
3. 탐지 좌표 융합 노드(`marker.py`)와 Reactive 주행 노드(`patrol.py`) 실행
4. RViz2와 Qt HUD에서 지도, 탐지 결과, 로봇 상태를 확인

장비별 셸 스크립트와 Qt HUD 소스는 팀원의 개인 Linux 환경에서 관리되었습니다. 이 저장소에는 제가 담당한 핵심 Python 노드와 학습·분석 자료를 공개합니다.

---

## ✨ 주요 기능

### 1. 실시간 SLAM 매핑
- LDS-02 LiDAR 기반 사전 지도 없이 실시간 점유 격자 지도 생성
- SLAM Toolbox (ROS2 Humble) 적용
- RViz2를 통한 실시간 맵 시각화

### 2. YOLO 기반 위협 객체 탐지
- 군인(class 0) / 전차(class 1) 실시간 탐지
- Knowledge Distillation으로 경량화된 YOLO11n 모델 사용
- NCNN 변환을 통해 라즈베리파이에서 5FPS 실시간 추론

### 3. SLAM-YOLO 융합 위협 지도 자동 생성
- TF2 좌표 변환으로 탐지 객체를 맵 좌표계에 등록
- XGBoost를 이용한 ML 거리추정으로 마커 정확도 향상
- 3회 확정 카운트 기반 오탐지 필터링
- 군인 → 초록 마커 / 전차 → 빨간 마커

### 4. 위협 연동 자율 회피 순찰
- 확정 위협 좌표 접근 시 자동 유턴 (40cm 이내)
- Visited Grid Map으로 미탐색 구역 우선 순찰
- 위험 구역 블랙리스트(penalty=50) 자동 등록

### 5. Qt 전술 HUD 통합 모니터링
- 레이더 스코프 / YOLO 카메라 / SLAM 맵 / 나침반 / 적 위치 통합 표시
- Teleop(수동 조작) 모드 전환 지원
- 배터리 잔량 실시간 모니터링

---

## 🤖 AI · Perception

### YOLO 학습 과정


| 단계 | 데이터셋 | Soldier mAP50 | Tank mAP50 | mAP50 |
|:---:|:---:|:---:|:---:|:---:|
| 1차 (Roboflow, 900장) | 외부 데이터 | 0.097 | 0.073 | 0.040 |
| 2차 (직접 수집, 525장) | 직접 촬영 | 0.835 | 0.895 | 0.865 |
| **KD 적용 (YOLO11n KD)** | **직접 촬영** | **0.862** | **0.917** | **0.889** |
| YOLO11m (참고용) | 직접 촬영 | 0.879 | 0.927 | 0.904 |

#### 수집 데이터
<img src="assets/yolo_data_img.png" width="991" height="753"/>

### Knowledge Distillation

YOLO11n(2.6M) 크기를 유지하면서 YOLO11m(20.1M) 수준의 성능을 달성하기 위해  
Knowledge Distillation을 적용했습니다.

```
KD Loss = α × YOLO Loss + (1-α) × Teacher Loss

- YOLO Loss   = Box Loss + Class Loss + DFL Loss
- Teacher Loss = Teacher Box Loss + Teacher Class Loss + Teacher DFL Loss
```

> **결과:** YOLO11n 대비 **2.4% mAP 향상**을 2.6M 파라미터 유지하면서 달성

### Depth Estimation (XGBoost)

Depth Camera의 부하·노이즈 문제를 해결하기 위해 ML 기반 거리 추정 모델을 구축했습니다.

- **데이터 수집:** Depth Camera + TurtleBot + Bounding Box 조합 650개
- **Feature (42개):** Bounding Box 좌표, 객체 크기, IMU, ratio 등
- **모델 비교:** Random Forest / Linear Regression / **XGBoost** / LightGBM
- **최종 모델:** XGBoost (Feature Importance 기반 불필요 feature 제거)
- **Test MAE: 3.8cm**

---

## 🗺️ Path Algorithm

### 장애물 회피
1. 전방 장애물 감지 → `escaping = True`
2. 좌우 거리 비교 → 여유 공간 방향으로 회전
3. 정면이 클리어되고 timeout 경과 → 탈출 완료

### 적군 탐지 & 유턴
1. YOLO 탐지 → `/detections` 발행
2. `detection_marker_node`에서 3회 확정 → `/danger_detected` 발행
3. `reactive_patrol_node`에서 위험 좌표 저장
4. 위험 좌표 40cm 이내 접근 시 유턴 시작
5. `STOP(3s) → ROTATE(5s) → ESCAPE(2s) → FORWARD(1s)` 순서로 회피

### 정상 주행 및 커버리지 최적화
1. 벽 보정 주행 (좌우 critical distance 기반)
2. Visited Grid Map으로 지나온 셀 카운트 기록
3. 갈림길에서 방문 횟수가 적은 방향 우선 선택
4. 위험 구역 주변 셀에 패널티(50) 부여 → 재접근 억제

---

## 🖥️ Qt 전술 HUD

<img src="assets/qt_hud.png" width="600"/>

**HUD 기능 목록**

| 기능 | 설명 |
|:---|:---|
| Login | 사용자 인증 및 접근 제어 |
| Radar Scope | LiDAR 기반 실시간 스캔 시각화 |
| Compass | IMU 기반 방향각 실시간 표시 |
| Camera | YOLO bbox 오버레이 실시간 영상 |
| LIDAR Map | SLAM 점유 격자 지도 표시 및 마커 |
| Enemy Info | 탐지 적 수 / 좌표 표시 |
| Battery State | 배터리 잔량 모니터링 |
| Manual Mode | Teleop 수동 조작 전환 |

---

## 📊 성능 지표

### Map 탐색 성능

| 지표 | Best Case | Average | 목표값 |
|:---:|:---:|:---:|:---:|
| 맵 탐색률 | **96.2%** | 73.4% | 75% |
| 마커 위치 오차 평균 | **10.5cm** | 22.3cm | 20~30cm |
| 객체 미탐지 수 | **1개** | 1.7개 | 1개 이하 |

### YOLO 성능 요약

| 모델 | 파라미터 | Soldier mAP50 | Tank mAP50 | mAP50 |
|:---:|:---:|:---:|:---:|:---:|
| YOLO11n | 2.6M | 0.835 | 0.895 | 0.865 |
| YOLO11n KD | 2.6M | 0.862 | 0.917 | **0.889** |
| YOLO11m | 20.1M | 0.879 | 0.927 | 0.904 |

### Depth Estimation
- **모델:** XGBoost
- **Test MAE:** 3.8cm

---

## 🔧 기술 스택

### System & Dev
![C++](https://img.shields.io/badge/C++-00599C?logo=cplusplus&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)
![VSCode](https://img.shields.io/badge/VSCode-007ACC?logo=visualstudiocode&logoColor=white)

### Hardware
| 부품 | 상세 |
|:---|:---|
| 로봇 플랫폼 | TurtleBot3 Burger |
| 온보드 컴퓨터 | Raspberry Pi 4 |
| 모터 컨트롤러 | OpenCR |
| LiDAR | LDS-02 (10Hz, ~220 points) |
| 카메라 | IMX219 PiCamera (640×480) |
| 구동 모터 | Dynamixel XL430 |

### Robotics & Navigation
![ROS2](https://img.shields.io/badge/ROS2-Humble-22314E?logo=ros)
- SLAM Toolbox, RViz2, Nav2, Gazebo

### AI & Perception
![YOLO](https://img.shields.io/badge/YOLO-v11-purple)
- Ultralytics YOLO11, NCNN (경량화), Roboflow (데이터셋)
- XGBoost (Depth Estimation)


### UI
- Qt5 (전술 HUD)

---

## 🚀 개발 환경 및 시스템 실행 흐름

### 환경 요구사항

```
- Ubuntu 22.04
- ROS2 Humble
- Python 3.10
- Raspberry Pi OS (Pi 4)
- ROS_DOMAIN_ID=11
- TURTLEBOT3_MODEL=burger
```

### 의존성 설치

```bash
# ROS2 Humble 설치 (Ubuntu PC)
sudo apt install ros-humble-slam-toolbox
sudo apt install ros-humble-tf2-geometry-msgs
sudo apt install ros-humble-visualization-msgs

# Python 패키지
pip install ultralytics joblib pandas xgboost scikit-learn --break-system-packages

# NCNN (Pi에서 설치)
pip install ncnn --break-system-packages
```

### 저장소 클론

```bash
git clone https://github.com/frogjun12-jpg/RADAR.git
cd RADAR
```

### 공개 코드 진입점

| 파일 | 역할 |
|---|---|
| `yolo_picamera_to_ubuntu_compressed_default.py` | Raspberry Pi 카메라 기반 YOLO 추론 및 ROS2 발행 |
| `marker.py` | 탐지 결과와 로봇 좌표를 융합해 지도 마커 생성 |
| `patrol.py` | 장애물·위협 구역을 반영한 Reactive 자율 순찰 |
| `Depth_Camera/` | Depth 데이터 수집 및 거리 추정 실험 |
| `YOLO/train/` | YOLO 학습 및 Knowledge Distillation 실험 |

전체 시스템은 TurtleBot3 bringup → YOLO 추론 → SLAM → 좌표 융합 → Reactive 주행 → 시각화 순서로 실행했습니다. 하드웨어 주소, 모델 경로, ROS2 토픽은 사용 환경에 맞게 조정해야 하며, 팀원 환경에서 사용한 셸 스크립트는 공개 저장소에 포함되어 있지 않습니다.

### 포트 설정 (Pi - 재시작 시마다 확인)

```bash
sudo chmod 666 /dev/ttyACM*
```

### 맵 저장

```bash
mkdir -p ~/maps
ros2 run nav2_map_server map_saver_cli -f ~/maps/patrol_map
```

---

## 📁 프로젝트 구조

```
RADAR/
├── patrol.py                         # Reactive 자율 순찰 노드
├── marker.py                         # YOLO-SLAM 좌표 융합 및 마커 노드
├── yolo_cam.py                       # Ubuntu 카메라 YOLO 노드
├── yolo_picamera_to_ubuntu_compressed_default.py
│                                       # Raspberry Pi YOLO 노드
├── Depth_Camera/                     # Depth 수집·분석 코드
├── DepthML/                          # 거리 추정 데이터와 모델 실험
├── YOLO/train/                       # YOLO 학습·KD 노트북
└── assets/                           # README 이미지
```

---

## 🔥 트러블 슈팅

### 1. Qt 카메라 딜레이
| | 내용 |
|:---|:---|
| **원인** | 압축되지 않은 이미지 스트림으로 인한 네트워크 대역폭 초과 |
| **이슈** | 카메라 지연이 다른 ROS2 노드 전체에 영향 |
| **해결** | `CompressedImage` 토픽 사용 (`/yolo_camera/compressed`, JPEG q=55) |

### 2. 주행 안정성
| | 내용 |
|:---|:---|
| **원인** | Reactive 알고리즘의 좁은 통로·모서리 처리 미흡 |
| **이슈** | 벽 모서리 충돌, 동일 경로 루프 반복 |
| **해결** | 파라미터 튜닝 (100회 이상 테스트 주행), 벽 모서리 물리 범퍼 추가, Visited Grid Map 도입 |

### 3. 마커 위치 오차
| | 내용 |
|:---|:---|
| **원인** | 로봇 회전 중 탐지 시 방향 오차 발생 |
| **이슈** | 20~30° 마커 위치 오차 |
| **해결** | `/robot_turning` 토픽으로 회전 중 마커 생성 차단, IMU + odom 이중 필터링 |

### 4. TF 변환 실패
| | 내용 |
|:---|:---|
| **원인** | 이미지 stamp 기반 TF 조회 시 미래 시간 오류 |
| **이슈** | `TransformException` 빈번 발생 |
| **해결** | `stamp=0` (latest TF) fallback 로직 추가, TF buffer cache 10초로 확장 |

---

## 🔭 향후 계획

### 달성 목표 (단기)
- [x] Real-time SLAM 구현
- [x] Qt GUI 응답성 개선
- [x] Perception 모델 고도화 (Knowledge Distillation)
- [ ] Qt GUI UX 추가 개선

### Future 목표 (장기)
- [ ] **Multi-SLAM** — 복수 로봇 협력 매핑
- [ ] **3D Mapping** — 3D LiDAR 기반 입체 지도
- [ ] **Visual SLAM** — 카메라 기반 SLAM
- [ ] **SDV 성능 향상** — 더 빠르고 안정적인 자율주행


---

<p align="center">
  <strong>RADAR</strong> — Risk Aware Detection And Recognition<br>
  Intel 9기 | 2026.04
</p>
