# 🚗 스테레오 비전과 레이더 기반 객체 거리 추정 및 안정화 기법 연구
## 📖 프로젝트 개요
자율주행 및 ADAS 환경에서 객체 인식뿐만 아니라, 객체까지의 **정확한 거리 추정**과 신호의 **시간적 안정성**은 안전 판단(TTC 등)에 핵심적인 요소입니다.
본 프로젝트는 **Stereo Vision**과 **Radar** 센서가 가지는 서로 다른 거리 추정 특성을 분석하고, 다양한 **신호 안정화 기법(Stabilization)**과 **센서 퓨전(Sensor Fusion)** 알고리즘을 적용하여 단일 센서 대비 향상된 거리 정확도와 안정성을 확보하는 것을 목표로 합니다.

---

## 🎯 주요 목표
1. **센서 비교 분석:** 동일 조건에서 Stereo Vision과 Radar의 거리 추정 성능 및 노이즈 특성 비교
2. **신호 안정화:** EMA, Kalman Filter, Outlier Gating 기법을 적용하여 거리 데이터의 Noise 및 Spike 제거
3. **센서 퓨전:** 거리 구간별(근거리/원거리) 특성에 따른 가중치 기반 퓨전(Weighted Fusion) 구현
4. **위험도 판단:** 안정화된 데이터를 기반으로 TTC(Time-to-Collision) 및 위험도(Safe/Warning/Danger) 시각화

---

## 🛠 사용 데이터 및 모델
* **Dataset:** Oxford Radar RobotCar Dataset
    * Stereo: Front Left/Right Camera 활용
    * Radar: SDK를 이용한 거리/방위각 데이터 추출
* **Object Detection Model:** YOLOv8s
    * Target Classes: car, bus, truck, person, motorcycle, bicycle, traffic light

---

## ⚙️ 시스템 아키텍처 및 작업 흐름 (Workflow)
1.  **전처리 (Preprocessing):** 타임스탬프 기준 프레임 정렬 및 센서 데이터 매칭
2.  **객체 탐지 (Object Detection):** YOLOv8s를 통한 프레임 내 객체 탐지
3.  **거리 추정 (Distance Estimation):**
    * Stereo: 좌우 카메라 매칭을 통한 거리 계산
    * Radar: 객체 위치에 매핑되는 레이더 포인트 추출
4.  **데이터 안정화 (Signal Stabilization):**
    * **EMA (Exponential Moving Average):** 가중 평균을 통한 신호 평활화
    * **Kalman Filter:** 예측과 측정의 가중 결합을 통한 최적 추정 및 노이즈 제거
    * **Outlier Gating + EMA:** MAD 기반 이상치 제거 후 EMA 적용
5.  **센서 퓨전 (Sensor Fusion):** 거리에 따른 동적 가중치 할당
6.  **위험도 판단 및 시각화:** 상대 속도 및 TTC 계산 후 Bounding Box 색상으로 위험 단계 표현

---

## 📊 실험 결과 및 분석

### 1. 거리 추정 안정화 (Stabilization) 결과
불안정한 거리 신호(Raw Data)에 대해 각 기법을 적용한 결과, 노이즈 감소 효과를 확인했으나 **반응성(Responsiveness)** 과의 Trade-off가 존재함을 확인했습니다.

| 기법 | 특징 | 장점 | 단점 |
| :--- | :--- | :--- | :--- |
| **EMA** | 이전 값과 현재 값의 가중 평균 | 구현 용이, 노이즈 감소 | 급격한 변화에 반응 느림 |
| **Kalman Filter** | 상태 예측 및 측정 업데이트 | 스파이크 완화, 매끄러운 시계열 생성 | 모델 파라미터(Q, R) 튜닝 필요 |
| **Outlier Gating** | MAD 기반 이상치 제거 | 가장 높은 안정성 수치 기록 | **과도한 필터링으로 위험 상황 인지 지연 발생** |

> **결론:** 과도한 안정화(Outlier Gating)는 실제 위험 상황에서의 반응 속도를 저하시킬 수 있어, 적절한 EMA 또는 Kalman Filter 튜닝이 더 효과적임.

### 2. 센서 퓨전 (Sensor Fusion) 결과
각 센서의 물리적 오차 특성을 고려한 **가중치 기반 퓨전(Weighted Fusion)** 을 적용하였습니다.

* **근거리 (~10m):** Stereo Vision의 해상도가 높으므로 Stereo 가중치 ↑
* **중·원거리 (>20m):** Stereo의 오차가 거리의 제곱에 비례하여 커지므로 Radar 가중치 ↑
* **결과:** 트럭 등 원거리 물체(50m 이상) 탐지 시, Stereo의 부정확한 거리(76.7m)를 Radar(53.5m)가 보정하여 퓨전 결과(59.7m)의 신뢰도 향상

---

## 💡 결론 (Conclusion)
* 단일 센서의 한계를 극복하기 위해 **거리 구간별 가중치를 달리하는 센서 퓨전** 전략이 유효함을 입증했습니다.
* 안정화 기법 적용 시, 단순히 노이즈를 줄이는 것보다 **안정성(Stability)과 반응성(Responsiveness)의 균형**을 맞추는 것이 ADAS 안전 판단에 필수적입니다.
