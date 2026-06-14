# 2026-06-14 Stage 1 region 16..20 성공 반경 축소 실험 정리

## 목적

Stage 1 camera-audit sweep에서 `15..25`가 `visible_learning_failed`로 분류된 뒤, 실패 원인이 perception이 아니라 마지막 중심 접근 precision gate인지 확인했다. 전체 25영역을 다시 돌리는 대신 region `16..20`을 focus run으로 나누고, 성공 반경을 `35 -> 25 -> 20 -> 15 -> 12mm`로 줄이는 ladder 방식으로 학습했다.

실제 MT4 하드웨어 motion은 실행하지 않았고 IsaacLab headless simulation만 사용했다.

## 이전 개선 사항과 순서

| 순서 | 개선/확인 | 결과 |
| ---: | --- | --- |
| 1 | Stage 0 workspace-entry 재학습 | task/runtime 포팅은 확인했지만 초기 workspace 진입 기준은 불안정했다. |
| 2 | reach-aware Stage 0, entry gate, standoff 조정 | 작업 박스 진입과 target 접근 보상을 분리했다. |
| 3 | Stage 1 9-cell plane | 35mm급 느슨한 성공 조건에서는 plane localization이 가능함을 확인했다. |
| 4 | Stage 1 25-cell 순차 curriculum | `1..14`는 mastered, `15..25`는 skipped로 병목이 재현됐다. |
| 5 | camera-audit 운영 영역 분리 | `1..14`는 operational, `15..25`는 카메라는 안정적이지만 학습 실패인 `visible_learning_failed`로 분리했다. |
| 6 | region 16/17 1차 focus run | `20mm`까지 가능, region `16`의 `15mm` 실패를 확인했다. |
| 7 | region 16..20 ladder 확장 | `18..20`은 `12mm`, `17`은 `15mm`, `16`은 `20mm`까지 통과했다. |

## 변경별 효과 비교

| 바꾼 것 | 좋아진 지표 | 나빠지거나 남은 지표 | 해석 |
| --- | --- | --- | --- |
| Stage 0 latch/gate 재검토 | 허위 성공 latch를 분리 | 초기 `inside_workspace_rate=0.0000` | 성공처럼 보이는 값을 그대로 믿으면 안 됨 |
| reach-aware entrygate | `workspace_entry_success_rate=0.4839`, `inside_workspace_rate=0.4856` | `center_1cm_rate=0.0000` | workspace entry는 회복, 정밀 grasp는 아님 |
| 9-cell sequential | `mastered_region_count=9` | final batch stability 부족 | curriculum 방식은 맞음 |
| 25-cell skip-stalled | `1..14` mastered, `15..25` skipped로 분리 | `15..25` success count `0` | 전체 반복보다 병목 분리가 필요 |
| camera audit | `camera_excluded=0/25` | `15..25`는 `visible_learning_failed` | perception보다 control/reward precision 병목 |
| 16..20 ladder | `18..20`은 `12mm` 통과 | `16`은 `15/12mm`, `17`은 `12mm` 실패 | final center/top-down XY 보강 대상이 `16/17`로 좁혀짐 |

## 코드/스크립트 변경

| 항목 | 내용 |
| --- | --- |
| `MT4_STAGE1_SUCCESS_RADIUS` | Stage 1 center/top-down XY 성공 반경을 환경변수로 조정 가능하게 했다. |
| `MT4_CENTER_SUCCESS_RADIUS` | center distance gate만 별도 override 가능하게 유지했다. |
| `MT4_TOP_DOWN_XY_SUCCESS_RADIUS` | top-down XY gate만 별도 override 가능하게 유지했다. |
| run name | `success35mm`, `success25mm`, `success20mm`, `success15mm`, `success12mm`처럼 반경이 run name에 남도록 했다. |
| `region_mastery.csv` | 성공한 샘플 중 center 기준 best score, distance, top-down XY, step을 함께 기록하게 했다. |
| `camera_workspace_audit.csv` | focus region의 visibility, camera region match, success count, operating status를 기록했다. |

## 학습 방법

공통 조건:

| 항목 | 값 |
| --- | --- |
| task | `Mirobot-Coordinate-Plane-Direct-v0` |
| num envs | `128` |
| requested max iterations | `1200` |
| seed | `42` |
| focus regions | `16`, `17`, `18`, `19`, `20` |
| tool-tip down offset | `0.035 m` |
| mastery target | focused region당 `10` successes |

실험 순서:

1. region `16`, `17`을 `35/25/20mm`로 확인했다.
2. region `16`의 `15mm`가 실패해 병목을 기록했다.
3. 사용자 결정에 따라 최종 목표를 `12mm`로 두고 region `16..20`을 같은 조건으로 확장했다.
4. region별 최소 통과 반경을 분리해서 기록했다.

## 최종 지표

| region | 35mm | 25mm | 20mm | 15mm | 12mm | 최종 상태 |
| ---: | --- | --- | --- | --- | --- | --- |
| 16 | mastered | mastered | mastered | failed | failed | `20mm`까지 가능 |
| 17 | mastered | mastered | mastered | mastered | failed | `15mm`까지 가능 |
| 18 | mastered | mastered | mastered | mastered | mastered | `12mm` 통과 |
| 19 | mastered | mastered | mastered | mastered | mastered | `12mm` 통과 |
| 20 | mastered | mastered | mastered | mastered | mastered | `12mm` 통과 |

핵심 run 수치:

| region | success radius | result | stop iteration | training time | best center distance | best top-down XY |
| ---: | ---: | --- | ---: | ---: | ---: | ---: |
| 16 | `20mm` | mastered, `10` successes | `178` | `154.35s` | `18.443mm` | `18.162mm` |
| 16 | `15mm` | not mastered, `0` successes | `1200` | `1041.89s` | n/a | n/a |
| 16 | `12mm` | not mastered, `0` successes | `1200` | `1079.70s` | n/a | n/a |
| 17 | `20mm` | mastered, `10` successes | `82` | `74.98s` | `19.084mm` | `15.563mm` |
| 17 | `15mm` | mastered, `10` successes | `304` | `260.71s` | `14.287mm` | `13.630mm` |
| 17 | `12mm` | not mastered, `0` successes | `1200` | `1061.65s` | n/a | n/a |
| 18 | `12mm` | mastered, `10` successes | `155` | `134.78s` | `10.822mm` | `10.758mm` |
| 19 | `12mm` | mastered, `10` successes | `236` | `203.68s` | `10.451mm` | `10.449mm` |
| 20 | `12mm` | mastered, `10` successes | `727` | `658.11s` | `11.299mm` | `11.289mm` |

Camera audit summary:

| region/run | target stereo | camera region match | mean estimate error | success count | operating status |
| --- | ---: | ---: | ---: | ---: | --- |
| 16, 15mm | `1.000000` | `1.000000` | `0.000000m` | `0` | pending |
| 16, 12mm | `1.000000` | `1.000000` | `0.000000m` | `0` | pending |
| 17, 15mm | `1.000000` | `1.000000` | `0.000000m` | `10` | operational |
| 17, 12mm | `1.000000` | `1.000000` | `0.000000m` | `0` | pending |
| 18, 12mm | `1.000000` | `1.000000` | `0.000000m` | `10` | operational |
| 19, 12mm | `1.000000` | `1.000000` | `0.000000m` | `10` | operational |
| 20, 12mm | `1.000000` | `1.000000` | `0.000000m` | `10` | operational |

## 분석

이번 결과는 `15..25` 실패가 camera exclusion 때문이라는 해석을 더 약하게 만든다. Focus run에서 target stereo visibility와 camera region match는 안정적이었고, 내부 좌표 기준 target estimate error도 `0.0m`로 기록됐다.

Region `18..20`은 `12mm`까지 통과했으므로 높은 z row나 끝 영역 전체가 불가능한 것은 아니다. 반면 region `16`은 `20mm` 이후 `15mm`에서 두 번 모두 실패했고, `12mm`도 실패했다. Region `17`은 `15mm`는 통과했지만 `12mm`에서 실패했다. 따라서 병목은 region별 final center/top-down XY precision과 action 안정성으로 보는 것이 맞다.

현재 운영 판단은 아래처럼 둔다.

| 기준 | 판단 |
| --- | --- |
| Stage 1 기본 운영 가능 영역 | 기존 `1..14` 유지 |
| region 16 연구 상태 | `20mm`까지 학습 가능, `15/12mm` 실패 |
| region 17 연구 상태 | `15mm`까지 학습 가능, `12mm` 실패 |
| region 18..20 연구 상태 | `12mm`까지 학습 가능 |
| 실제 운영 승격 | isolated focus 결과만으로는 보류 |
| 실패 원인 | camera exclusion이 아니라 마지막 center/top-down XY precision 병목 |

## 제안 사항

1. Region `16`부터 final 25mm 이내 center reward와 top-down XY reward를 더 가파르게 만든다.
2. Region `16`은 `15mm`를 먼저 통과시킨 뒤 `12mm`로 줄인다.
3. Region `17`은 같은 보강 조건으로 `12mm`만 재시도한다.
4. Region `16/17`이 `12mm`에 도달하면 `16..20` 전체 묶음 재검증을 수행한다.
5. 실제 MT4 motion은 여전히 Safety Gate 뒤에 둔다.

## 산출물

| 항목 | 경로 |
| --- | --- |
| 16 / 20mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-37-58_mt4_coordinate_plane_seq25_10success_success20mm_tipdown35mm_region16_128env_1200iter` |
| 16 / 15mm retry | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_20-07-50_mt4_coordinate_plane_seq25_10success_success15mm_tipdown35mm_region16_128env_1200iter_batch16to20` |
| 16 / 12mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_20-45-51_mt4_coordinate_plane_seq25_10success_success12mm_tipdown35mm_region16_128env_1200iter_batch16to20` |
| 17 / 15mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_20-25-50_mt4_coordinate_plane_seq25_10success_success15mm_tipdown35mm_region17_128env_1200iter_batch16to20` |
| 17 / 12mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_21-04-51_mt4_coordinate_plane_seq25_10success_success12mm_tipdown35mm_region17_128env_1200iter_batch16to20` |
| 18 / 12mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_21-22-51_mt4_coordinate_plane_seq25_10success_success12mm_tipdown35mm_region18_128env_1200iter_batch16to20` |
| 19 / 12mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_21-25-51_mt4_coordinate_plane_seq25_10success_success12mm_tipdown35mm_region19_128env_1200iter_batch16to20` |
| 20 / 12mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_21-29-51_mt4_coordinate_plane_seq25_10success_success12mm_tipdown35mm_region20_128env_1200iter_batch16to20` |
| local launch logs | `training_logs/launch/` |

원시 launch log는 크기가 커서 git에는 올리지 않고, 요약 분석과 run directory 경로만 기록한다.

## English Notes

Focused Stage 1 ladder runs show that regions `18`, `19`, and `20` can be mastered at a `12mm` success radius. Region `17` reaches `15mm` but fails at `12mm`, while region `16` reaches `20mm` but fails at both `15mm` and `12mm`. The bottleneck is final precision control/reward shaping rather than perception.
