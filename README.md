# RealSense Depth-Based Pick Detection Pipeline

Depth 이미지 기반으로 ROI 내 최상단 물체를 검출하고, 월드 좌표(mm) 및 Gripper Yaw 각도를 계산하는 Grasp 후보 추출 시스템입니다.  
Intel RealSense를 사용하며 Median Depth Snapshot + Distance Transform + PCA 기반 방향 추정을 포함합니다.

---

## Features

- Median depth snapshot 기반 노이즈 감소
- Top-percent depth segmentation
- Morphology 기반 마스크 정제
- Blob scoring (area + compactness + distance peak)
- Distance Transform 기반 안전 픽 포인트 선정
- PCA 기반 gripper yaw 계산
- Pixel → World 좌표 변환 (Extrinsic 적용)
- mm 단위 ROS 연동 payload 출력

---

## Project Structure

```
.
├── main.py
├── camera_loop.py
├── camera_events.py
├── snapshot.py
├── pick_pipeline.py
├── coordinate.py
├── render.py
└── camcalib.npz
```

---

## Requirements

- Python 3.8+
- numpy
- opencv-python
- pyrealsense2
- Intel RealSense D4xx series

Install:

```
pip install numpy opencv-python pyrealsense2
```

---

## Run

```
python main.py
```

Controls:

- Left Click : Depth sampling
- SPACE      : Object detection
- r          : Reset clicks
- q / ESC    : Quit

---

## Output

Console (mm 단위):

```
flat_clicked_xy_mm = [[X_mm, Y_mm], ...]
xyz_angle_mm = [X_mm, Y_mm, Zapp_mm, yaw_deg]
```

Snapshot image:

```
result/color.jpg
```

---

## Coordinate System

- 내부 계산 단위: cm
- 최종 출력 단위: mm
- yaw 범위: -90° ~ +90° (y-axis 기준)
- 접근 높이: Z + 3cm

---

## Detection Logic

1. Depth filtering (0.20m ~ 2.00m)
2. ROI crop
3. Top 3% percentile depth segmentation
4. MAD 기반 adaptive band
5. Morphology close → open
6. Connected component selection
7. Distance transform
8. 중심성 가중 픽 포인트 선택
9. PCA 기반 yaw 계산
10. Pixel → World 변환

---

## Failure Conditions

Detection 실패 조건:

- 유효 depth 비율 부족
- 최소 blob area 미달
- Distance peak 부족
- PCA anisotropy ratio 미달
- ROI edge 근접 + 낮은 접근 높이

---

## ROS Integration

camera_loop.py 내부 payload:

```
self.last_payload = {
    "ok": bool,
    "color_path": str,
    "flat_clicked_xy": [[X_mm, Y_mm], ...],
    "xyz_angle": [X_mm, Y_mm, Zapp_mm, yaw_deg]
}
```

ROS 노드에서 직접 사용 가능.

---

## Calibration

camcalib.npz 필수 포함 항목:

- camera_matrix
- dist_coeffs
- T_cam_to_work

Pixel → World 변환에 사용됩니다.
