# 2026-06-14 Stage 1 region 16/17 성공 반경 축소 실험 정리

## 목적

Stage 1 camera-audit sweep에서 `15..25`가 `visible_learning_failed`로 분류된 뒤, 실패 원인이 perception이 아니라 마지막 중심 접근 precision gate인지 확인했다. 전체 25영역을 다시 돌리는 대신 region `16`, `17`만 focus run으로 나누고, 성공 반경을 큰 값에서 작은 값으로 줄이는 ladder 방식으로 재학습했다.

실제 MT4 하드웨어 motion은 실행하지 않았고 IsaacLab headless simulation만 사용했다.

## 이전 개선 사항과 순서

| 순서 | 개선/확인 | 결과 |
| ---: | --- | --- |
| 1 | Stage 0 workspace-entry 재학습 | task/runtime 포팅은 확인했지만 초기 workspace 진입 기준은 불안정했다. |
| 2 | reach-aware Stage 0, entry gate, standoff 조정 | 작업 박스 진입과 target 접근 보상을 분리했다. |
| 3 | Stage 1 9-cell plane | 35mm급 느슨한 성공 조건에서는 plane localization이 가능함을 확인했다. |
| 4 | Stage 1 25-cell 순차 curriculum | `1..14`는 mastered, `15..25`는 skipped로 병목이 재현됐다. |
| 5 | camera-audit 운영 영역 분리 | `1..14`는 operational, `15..25`는 카메라는 안정적이지만 학습 실패인 `visible_learning_failed`로 분리했다. |
| 6 | region 16/17 focus run | 높은 z/끝 영역을 전체 sweep에서 분리해 성공 반경 ladder로 확인했다. |

## 코드/스크립트 변경

| 항목 | 내용 |
| --- | --- |
| `MT4_STAGE1_SUCCESS_RADIUS` | Stage 1 center/top-down XY 성공 반경을 환경변수로 조정 가능하게 했다. |
| `MT4_CENTER_SUCCESS_RADIUS` | center distance gate만 별도 override 가능하게 유지했다. |
| `MT4_TOP_DOWN_XY_SUCCESS_RADIUS` | top-down XY gate만 별도 override 가능하게 유지했다. |
| run name | `success35mm`, `success25mm`, `success20mm`처럼 반경이 run name에 남도록 했다. |
| `region_mastery.csv` | 성공한 샘플 중 center 기준 best score, distance, top-down XY, step을 함께 기록하게 했다. |
| active region marker | GUI/영상 확인 시 현재 시도 영역과 성공 위치를 구분하기 위한 marker를 추가했다. |

## 학습 방법

공통 조건:

| 항목 | 값 |
| --- | --- |
| task | `Mirobot-Coordinate-Plane-Direct-v0` |
| num envs | `128` |
| requested max iterations | `1200` |
| seed | `42` |
| focus regions | `16`, `17` |
| tool-tip down offset | `0.035 m` |
| mastery target | focused region당 `10` successes |

실험 순서:

1. region `16`, `17`을 성공 반경 `35mm`로 각각 실행한다.
2. 둘 다 mastered가 되면 `25mm`로 줄인다.
3. 다시 둘 다 mastered가 되면 `20mm`로 줄인다.
4. `15mm`는 먼저 region `16`으로 probe한다.
5. 실패하면 같은 조건으로 region `17`을 계속 돌리기보다 reward/gate 조정을 우선 검토한다.

## 최종 지표

| region | success radius | result | stop iteration | training time | best center distance | best top-down XY |
| ---: | ---: | --- | ---: | ---: | ---: | ---: |
| 16 | `35mm` | mastered, `11` successes | `19` | `16.89s` | `34.645mm` | `7.158mm` |
| 17 | `35mm` | mastered, `11` successes | `25` | `23.37s` | `33.052mm` | `23.117mm` |
| 16 | `25mm` | mastered, `10` successes | `134` | `116.11s` | `23.272mm` | `21.975mm` |
| 17 | `25mm` | mastered, `10` successes | `40` | `35.42s` | `23.989mm` | `9.244mm` |
| 16 | `20mm` | mastered, `10` successes | `178` | `154.35s` | `18.443mm` | `18.162mm` |
| 17 | `20mm` | mastered, `10` successes | `82` | `74.98s` | `19.084mm` | `15.563mm` |
| 16 | `15mm` | not mastered, `0` successes | `1200` | `1078.35s` | n/a | n/a |

Camera audit summary:

| region/run | target stereo | camera region match | mean estimate error | target gripper-camera visible |
| --- | ---: | ---: | ---: | ---: |
| 16, 20mm | `1.000000` | `1.000000` | `0.000000m` | `0.790426` |
| 17, 20mm | `1.000000` | `1.000000` | `0.000000m` | `0.618259` |
| 16, 15mm | `1.000000` | `1.000000` | `0.000000m` | `0.903113` |

## 분석

이번 결과는 `16`, `17`이 perception 때문에 실패한 영역이 아니라는 판단을 더 강하게 만든다. 두 영역 모두 target stereo visibility와 camera region match가 안정적으로 `1.0`이고, 내부 좌표 기준 target estimate error도 `0.0m`로 기록됐다.

성공 반경 ladder에서는 `20mm`까지 region `16`, `17` 모두 mastered됐다. 다만 `20mm` region `16`은 `178` iteration, region `17`은 `82` iteration이 필요해 `35mm`, `25mm`보다 난도가 분명히 높아졌다. `15mm` region `16`은 전체 `1200` iteration을 끝까지 돌았지만 성공이 0회였으므로 현재 reward/action 조건에서는 15mm gate를 바로 닫기 어렵다.

따라서 현재 운영 판단은 아래처럼 둔다.

| 기준 | 판단 |
| --- | --- |
| Stage 1 운영 가능 영역 | 기존 `1..14` 유지 |
| region 16/17 연구 상태 | `20mm`까지 학습 가능 확인 |
| 실제 운영 성공 반경 후보 | 당장은 `20mm`가 현실적 |
| 다음 precision 목표 | `15mm` 직행보다 `18mm` 또는 reward shaping 후 `15mm` |
| 실패 원인 | camera exclusion이 아니라 마지막 center/top-down XY precision 병목 |

## 제안 사항

1. 다음 run은 `15mm` 반복보다 `18mm` 중간 단계를 먼저 둔다.
2. `15mm`를 목표로 할 때는 성공 반경만 줄이지 말고, 마지막 25mm 이내 구간의 center reward와 top-down XY reward를 더 가파르게 만든다.
3. region `16`에서 먼저 `18mm -> 15mm`를 열고, 통과한 뒤 region `17`로 복제한다.
4. 전체 `15..25` 재시도는 region `16/17`에서 15mm 성공이 나온 뒤에 수행한다.
5. 실제 MT4 motion은 여전히 Safety Gate 뒤에 둔다.

## 산출물

| 항목 | 경로 |
| --- | --- |
| 16 / 35mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-33-49_mt4_coordinate_plane_seq25_10success_success35mm_tipdown35mm_region16_128env_1200iter` |
| 17 / 35mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-34-41_mt4_coordinate_plane_seq25_10success_success35mm_tipdown35mm_region17_128env_1200iter` |
| 16 / 25mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-35-11_mt4_coordinate_plane_seq25_10success_success25mm_tipdown35mm_region16_128env_1200iter` |
| 17 / 25mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-37-15_mt4_coordinate_plane_seq25_10success_success25mm_tipdown35mm_region17_128env_1200iter` |
| 16 / 20mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-37-58_mt4_coordinate_plane_seq25_10success_success20mm_tipdown35mm_region16_128env_1200iter` |
| 17 / 20mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-40-40_mt4_coordinate_plane_seq25_10success_success20mm_tipdown35mm_region17_128env_1200iter` |
| 16 / 15mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-42-03_mt4_coordinate_plane_seq25_10success_success15mm_tipdown35mm_region16_128env_1200iter` |
| local launch logs | `training_logs/launch/` |

원시 launch log는 크기가 커서 git에는 올리지 않고, 요약 분석과 run directory 경로만 기록한다.

## English Notes

Focused Stage 1 runs show that regions `16` and `17` can be mastered down to a `20mm` success radius with stable camera audit metrics. Region `16` fails at `15mm` after the full `1200` iterations, so the current bottleneck is final precision control/reward shaping rather than perception.
