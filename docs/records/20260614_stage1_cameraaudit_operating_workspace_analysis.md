# 2026-06-14 Stage 1 camera-audit 운영 가동 영역 결과

## 목적

2026-06-14 설계 계획에 따라 Stage 1 5x5 plane 학습을 다시 실행하고, 학습 성공 여부와 카메라 추정 안정성을 함께 기록했다. 목표는 전체 25개 영역을 억지로 모두 성공시키는 것이 아니라, 실제 운영 기준으로 쓸 수 있는 영역과 카메라는 안정적인데 접근/제어가 실패한 영역을 분리하는 것이다.

실제 MT4 하드웨어 motion은 실행하지 않았고 IsaacLab headless simulation만 사용했다.

## 코드/스크립트 변경

| 항목 | 내용 |
| --- | --- |
| `camera_workspace_audit.csv` | run directory에 영역별 카메라/학습 audit CSV 생성 |
| `operating_status` | `operational`, `visible_learning_failed`, `camera_excluded`, `pending` 분류 |
| TensorBoard counters | operational/visible_learning_failed/camera_excluded/pending 영역 수 기록 |
| sweep script | run name을 `cameraaudit`로 구분하고 audit 기준값 출력 |

기본 camera audit 기준:

| 항목 | 값 |
| --- | ---: |
| min samples | `256` |
| target stereo visibility threshold | `0.95` |
| camera region match threshold | `0.95` |
| max target estimate error | `0.005 m` |

## 학습 실행

| 항목 | 값 |
| --- | --- |
| task | `Mirobot-Coordinate-Plane-Direct-v0` |
| run name | `mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter` |
| run dir | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter` |
| num envs | `128` |
| requested max iterations | `3000` |
| actual stop | environment requested stop at iteration `1880` |
| seed | `42` |
| plane shape | `5x5`, regions `1..25` |
| tool-tip down offset | `0.035 m` |
| center success radius | `0.012 m` |
| stall skip limit | `3840` env steps without new success |
| latest checkpoint | `model_1880.pt` |

Command:

```bash
./scripts/train_mirobot_coordinate_stage1_plane_sweep25_skip.sh
```

## 최종 지표

| metric | final |
| --- | ---: |
| operational regions | `14/25` |
| visible learning failed regions | `11/25` |
| camera excluded regions | `0/25` |
| mastered regions | `14/25` |
| skipped regions | `11/25` |
| final checkpoint | `model_1880.pt` |

Region mastery:

| region range | result |
| --- | --- |
| `1..14` | mastered |
| `15..25` | skipped: `no_new_success_for_3840_env_steps` |

Camera operating status:

| region range | status |
| --- | --- |
| `1..14` | `operational` |
| `15..25` | `visible_learning_failed` |
| none | `camera_excluded` |

영역별 성공 횟수:

| region | success_count | operating_status |
| ---: | ---: | --- |
| 1 | 14 | operational |
| 2 | 18 | operational |
| 3 | 24 | operational |
| 4 | 12 | operational |
| 5 | 14 | operational |
| 6 | 11 | operational |
| 7 | 25 | operational |
| 8 | 21 | operational |
| 9 | 21 | operational |
| 10 | 18 | operational |
| 11 | 10 | operational |
| 12 | 34 | operational |
| 13 | 39 | operational |
| 14 | 11 | operational |
| 15-25 | 0 | visible_learning_failed |

## 분석

이번 run은 이전 2026-06-13 skip-stalled sweep과 같은 결론을 재현하면서, 실패 이유를 더 명확히 분리했다. `1..14`는 mastered이면서 camera audit도 통과했으므로 현재 Stage 1 기준 운영 가능 영역으로 볼 수 있다.

`15..25`는 모두 skipped지만 camera audit에서는 target stereo visibility `1.0`, camera region match `1.0`, mean target estimate error `0.0`으로 기록됐다. 따라서 이 영역들은 카메라에서 안 보이거나 영역 추정이 틀린 문제가 아니다. 현재 실패 원인은 마지막 중심 접근과 top-down lateral/height gate, 특히 `center_success_radius=0.012 m`를 닫지 못하는 제어/reward 병목으로 보는 것이 맞다.

좌표상 `15`는 중간 높이 `z=0.068`, `y=+0.038` 끝 영역이고, `16..25`는 더 높은 두 줄 `z=0.079`, `z=0.090`이다. 즉 현재 운영 영역은 낮은 두 줄과 중간 높이의 일부(`1..14`)까지이며, `+y` 끝 또는 높은 z 영역부터 precision 성공 조건이 깨진다.

## 다음 판단

1. 실제 운영 workspace는 일단 `1..14`로 제한한다.
2. `15..25`는 perception 제외가 아니라 `visible_learning_failed`로 두고, focus run에서 접근축/정밀 중심 reward를 따로 조정한다.
3. 다음 코드 실험은 전체 sweep 반복보다 region `15`, `16`, `18`, `21` focus run이 더 효율적이다.
4. 실제 MT4 motion은 여전히 Safety Gate 뒤에 둔다.

## 산출물

| 항목 | 경로 |
| --- | --- |
| run dir | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter` |
| latest checkpoint | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter/model_1880.pt` |
| region mastery | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter/region_mastery.csv` |
| camera workspace audit | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter/camera_workspace_audit.csv` |
| workspace cells | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter/workspace_reach_limited_25_cells.md` |

## English Notes

The camera-audit sweep confirms `1..14` as operational Stage 1 regions. Regions `15..25` are camera-stable but learning-failed, so they should be treated as control/reward bottlenecks rather than perception failures.
