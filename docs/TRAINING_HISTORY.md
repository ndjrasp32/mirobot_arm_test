# MT4 누적 학습 기록

Date: 2026-06-14 KST

## 현재 결론

Stage 1 camera-aligned 5x5 plane은 기본 운영 영역을 `1..14`로 둔다. `15..25`는 카메라에서 제외된 영역이 아니라 `visible_learning_failed` 영역이다.

2026-06-14 focus ladder에서 region `16..20`을 성공 반경 `35 -> 25 -> 20 -> 15 -> 12mm`로 줄여 확인했다. Region `18..20`은 최종 목표인 `12mm`까지 mastered됐다. Region `17`은 `15mm`까지 mastered됐지만 `12mm`는 실패했고, region `16`은 `20mm`까지 mastered된 뒤 `15mm`, `12mm`에서 실패했다.

## 단계별 누적 요약

| 날짜 | 단계 | 목적 | 결과 | 상세 기록 |
| --- | --- | --- | --- | --- |
| 2026-06-10 | Stage 0 workspace-entry | MT4 coordinate task/runtime 포팅 확인 | runtime은 통과, `inside_workspace_rate=0.0000` | `docs/records/20260610_stage0_workspace_entry_result.md` |
| 2026-06-10 | Stage 0 gate fix | workspace 바깥 근접 성공 latch 방지 | `workspace_entry_success_radius` 기준 조정 | `docs/records/20260610_stage0_gate_fix.md` |
| 2026-06-10 | Stage 0 retrain | stricter entry gate 재학습 | 안정 성공 미완, reward/gate 재검토 필요 | `docs/records/20260610_stage0_workspace_entry_retrain.md` |
| 2026-06-11 | Stage 0 phase split | 접근 단계를 분리 | top-down/reach reward 분리 필요 확인 | `docs/records/20260611_phase_split_stage0_analysis.md` |
| 2026-06-11 | Stage 0 relaxed | gate 완화 | 일부 접근 개선, handoff 정책은 미완 | `docs/records/20260611_phase_split_relaxed_stage0_analysis.md` |
| 2026-06-11 | Stage 0 reach-aware | reach-aware post-latch/standoff/entry gate | 최신 Stage 0 근거, Stage 1로 실험 진행 가능 | `docs/records/20260611_reach_aware_stage0_entrygate_600iter_analysis.md` |
| 2026-06-11 | Stage 1 3x3/plane | 35mm급 plane localization 확인 | 완화 조건에서 plane 일부 성공 | `docs/records/20260611_stage1_plane_xy035_center035_analysis.md` |
| 2026-06-12 | Stage 1 seq9 | 9영역 순차 curriculum | `mastered_region_count=9`, stability는 추가 확인 필요 | `docs/records/20260612_stage1_seq9_5success_analysis.md` |
| 2026-06-12 | Stage 1 seq25 tipdown35 | 25영역 확장 | 16 이후 병목 재현 | `docs/records/20260612_stage1_seq25_tipdown35_analysis.md` |
| 2026-06-12 | Stage 1 focus | region 7/9 접근축 분석 | 작업 박스를 arm 쪽 10mm 이동, target을 35mm 낮추는 방향 확정 | `docs/records/20260612_stage1_region7_9_focus_axis_analysis.md` |
| 2026-06-12 | Stage 1 region17 continue | 단순 추가 학습으로 병목 해소 여부 확인 | 실패, center precision 병목 판단 | `docs/records/20260612_stage1_region17_continue_failure_record.md` |
| 2026-06-13 | Stage 1 seq25 rerun | 128env rerun | region progression 기록 개선 필요 확인 | `docs/records/20260613_stage1_seq25_128env_rerun_analysis.md` |
| 2026-06-13 | Stage 1 skip-stalled | 막힌 영역을 skip하고 전체 상태 기록 | `1..14` mastered, `15..25` skipped | `docs/records/20260613_stage1_seq25_skipstalled3840_analysis.md` |
| 2026-06-14 | Stage 1 camera audit | 학습 실패와 카메라 실패 분리 | `1..14` operational, `15..25` visible_learning_failed, camera_excluded `0` | `docs/records/20260614_stage1_cameraaudit_operating_workspace_analysis.md` |
| 2026-06-14 | Stage 1 16..20 radius ladder | 성공 반경 축소로 precision 한계 확인 | `18..20`은 `12mm`, `17`은 `15mm`, `16`은 `20mm`까지 mastered | `docs/records/20260614_stage1_regions16_20_radius_ladder_analysis.md` |

## 최신 Stage 1 Camera-Audit 결과

| 구분 | 결과 |
| --- | --- |
| run | `2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter` |
| actual stop | iteration `1880` |
| final checkpoint | `model_1880.pt` |
| mastered regions | `14/25` |
| operational regions | `1..14` |
| visible_learning_failed | `15..25` |
| camera_excluded | `0/25` |
| 판단 | 실패 원인은 perception exclusion이 아니라 final center/top-down precision 병목 |

## Region 16..20 성공 반경 Ladder

| region | 35mm | 25mm | 20mm | 15mm | 12mm | 현재 판단 |
| ---: | --- | --- | --- | --- | --- | --- |
| 16 | mastered | mastered | mastered | failed | failed | 20mm까지 가능, 최우선 병목 |
| 17 | mastered | mastered | mastered | mastered | failed | 12mm 재시도 대상 |
| 18 | mastered | mastered | mastered | mastered | mastered | 12mm 통과 |
| 19 | mastered | mastered | mastered | mastered | mastered | 12mm 통과 |
| 20 | mastered | mastered | mastered | mastered | mastered | 12mm 통과 |

핵심 stop iteration:

| region | radius | result | stop iteration | training time |
| ---: | ---: | --- | ---: | ---: |
| 16 | `15mm` | failed, `0` successes | `1200` | `1041.89s` |
| 16 | `12mm` | failed, `0` successes | `1200` | `1079.70s` |
| 17 | `15mm` | mastered, `10` successes | `304` | `260.71s` |
| 17 | `12mm` | failed, `0` successes | `1200` | `1061.65s` |
| 18 | `12mm` | mastered, `10` successes | `155` | `134.78s` |
| 19 | `12mm` | mastered, `10` successes | `236` | `203.68s` |
| 20 | `12mm` | mastered, `10` successes | `727` | `658.11s` |

## 학습 순서 기준

현재 권장 순서는 아래와 같다.

1. Stage 0 workspace-entry/reach-aware smoke로 runtime과 관측 구성을 확인한다.
2. Stage 1 5x5 plane에서 camera-audit와 region_mastery를 같이 기록한다.
3. 실패 영역은 전체 sweep 반복 전에 focus region으로 분리한다.
4. Focus region에서 성공 반경 ladder를 `35 -> 25 -> 20 -> 15 -> 12mm` 순서로 돌린다.
5. Region별 최소 성공 반경을 분리해서 기록한다.
6. `12mm` 실패 영역은 단순 반복보다 final center/top-down XY reward와 action 안정성을 보강한다.
7. Stage 2 5x5x4 volume은 Stage 1 precision이 안정된 뒤에 연다.

## 해석

Region `15..25`는 카메라 target stereo visibility, camera region match, target estimate error 관점에서 제외할 근거가 없다. 최신 focus 결과도 같은 방향이다. Region `18..20`은 `12mm`까지 통과했으므로 상단/끝 영역 전체가 불가능한 것은 아니다. 병목은 region `16`과 `17`의 마지막 center/top-down XY precision이다.

따라서 실제 운영 workspace는 당분간 `1..14`로 제한하고, region `16..20`은 연구 영역으로 둔다. `16/17`이 `12mm`를 통과한 뒤에만 `16..20` 묶음 운영 승격을 검토한다.
