

````markdown
# Scout Mini ROS2 설치 및 실행 가이드 (오류/수정 포함)

이 문서는 Scout Mini ROS2와 UGV SDK를 설치하고 실행하는 과정에서 발생할 수 있는 오류와 그 해결 방법까지 포함한 가이드입니다.

---

## 1. ROS2 워크스페이스 생성

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
````

---

## 2. 패키지 클론

```bash
git clone https://github.com/agilexrobotics/ugv_sdk.git
git clone https://github.com/agilexrobotics/scout_ros2.git
git clone -b ros2 https://github.com/Slamtec/rplidar_ros.git
```

---

## 3. Scout Base 코드 수정

빌드 시 오류가 발생하여 다음 파일들을 수정함:

### 3-1. scout_base/src/scout_base_ros.cpp

* 기존: ROS1 스타일 `catkin_install_in_prefix_root` 등 불필요 변수 존재
* 수정: `find_package(rclcpp REQUIRED)` 등 ROS2 스타일로 변경
* C++14 컴파일 옵션 명시, 필요 없는 옵션 제거

> ⚠️ 빌드 오류 원인
>
> * ROS2에서는 일부 ROS1 헤더가 호환되지 않음
> * CMakeLists.txt에서 ROS2 패키지 찾기 방식 필요
> * shared_ptr 관련 객체 수명 관리 문제 → `std::shared_ptr` 명시적으로 사용

### 3-2. scout_base/CMakeLists.txt


* 기존: `#include "tf2_geometry_msgs/tf2_geometry_msgs.h"`
* 수정: `#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"`
* `std::make_shared<ScoutBaseRos>` 대신 `shared_from_this()` 사용 시 주의

---

## 4. colcon 빌드

```bash
cd ~/ros2_ws
colcon build --symlink-install
```

> ⚠️ 흔히 발생한 문제
>
> 1. **CMake 경고**
>
>    * `#warning This header is obsolete` → 수정으로 해결
> 2. **Segmentation fault / 빌드 실패**
>
>    * ROS2 환경 미설정 → `source ~/ros2_ws/install/setup.bash`
> 3. **ugv_sdk ‘this’ pointer null**
>
>    * 샘플 코드 경고, 실제 실행에는 영향 없음

---

## 5. CAN 장치 활성화

```bash
sudo ip link set can0 up type can bitrate 500000
```

> ⚠️ 오류 및 해결
>
> * `Device not found` → CAN 장치 이름 확인
> * 비트레이트 오류 → 하드웨어 지원 비트레이트 확인

---

## 6. Scout Base Node 실행

```bash
ros2 run scout_base scout_base_node --ros-args -p port_name:=can0 -p is_scout_mini:=true
```

> ⚠️ 오류 및 해결
>
> * `Detected protocol: UNKNOWN` → CAN 케이블, 포트, 전원 확인
> * `cannot publish data` / `Segmentation fault` → ROS2 환경 설정 (`source ~/ros2_ws/install/setup.bash`)

---

## 7. Teleoperation (원격 조종)

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

> ⚠️ 오류 및 해결
>
> * `Package 'teleop_twist_keyboard' not found`
>
>   ```bash
>   sudo apt install ros-humble-teleop-twist-keyboard
>   ```

---

## 8. 요약

### 설치/실행 순서

1. 워크스페이스 생성
2. 패키지 클론
3. scout_base cpp 및 CMakeLists.txt 수정
4. `colcon build --symlink-install`
5. CAN 장치 활성화
6. Scout Base Node 실행
7. Teleop 키보드 실행

### 주의 사항

* CAN 연결, 포트 설정, 전원 확인 필수
* 빌드 경고는 대부분 수정으로 해결
* `teleop_twist_keyboard` 패키지 설치 필수

```

