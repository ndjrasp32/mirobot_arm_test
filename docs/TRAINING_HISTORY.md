# MT4 누적 학습 기록

Date: 2026-06-14 KST

## 현재 결론

Stage 1 camera-aligned 5x5 plane은 `1..14`를 운영 가능 영역으로 본다. `15..25`는 카메라에서 제외된 영역이 아니라 `visible_learning_failed` 영역이다. Region `16`, `17` focus run은 성공 반경 `20mm`까지 mastered됐고, region `16`의 `15mm`는 실패했다.

다음 실험은 `35mm -> 25mm -> 20mm -> 15mm` 직행이 아니라 `20mm -> 18mm -> 15mm`와 final precision reward 보강이다.

## 단계별 누적 요약

| 날짜 | 단계 | 목적 | 결과 | 상세 기록 |
| --- | --- | --- | --- | --- |
| 2026-06-10 | Stage 0 workspace-entry | MT4 coordinate task/runtime 포팅 확인 | runtime은 통과, `inside_workspace_rate=0.0000` | `docs/records/training/20260610_stage0_workspace_entry_result.md` |
| 2026-06-10 | Stage 0 gate fix | workspace 바깥 근접 성공 latch 방지 | `workspace_entry_success_radius` 기준 조정 | `docs/records/training/20260610_stage0_gate_fix.md` |
| 2026-06-10 | Stage 0 retrain | stricter entry gate 재학습 | 안정 성공 미완, reward/gate 재검토 필요 | `docs/records/training/20260610_stage0_workspace_entry_retrain.md` |
| 2026-06-11 | Stage 0 phase split | 접근 단계를 분리 | top-down/reach reward 분리 필요 확인 | `docs/records/training/20260611_phase_split_stage0_analysis.md` |
| 2026-06-11 | Stage 0 relaxed | gate 완화 | 일부 접근 개선, handoff 정책은 미완 | `docs/records/training/20260611_phase_split_relaxed_stage0_analysis.md` |
| 2026-06-11 | Stage 0 reach-aware | reach-aware post-latch/standoff/entry gate | 최신 Stage 0 근거, Stage 1로 실험 진행 가능 | `docs/records/training/20260611_reach_aware_stage0_entrygate_600iter_analysis.md` |
| 2026-06-11 | Stage 1 3x3/plane | 35mm급 plane localization 확인 | 완화 조건에서 plane 일부 성공 | `docs/records/training/20260611_stage1_plane_xy035_center035_analysis.md` |
| 2026-06-12 | Stage 1 seq9 | 9영역 순차 curriculum | `mastered_region_count=9`, stability는 추가 확인 필요 | `docs/records/training/20260612_stage1_seq9_5success_analysis.md` |
| 2026-06-12 | Stage 1 seq25 tipdown35 | 25영역 확장 | 16 이후 병목 재현 | `docs/records/training/20260612_stage1_seq25_tipdown35_analysis.md` |
| 2026-06-12 | Stage 1 focus | region 7/9 접근축 분석 | 작업 박스를 arm 쪽 10mm 이동, target을 35mm 낮추는 방향 확정 | `docs/records/training/20260612_stage1_region7_9_focus_axis_analysis.md` |
| 2026-06-12 | Stage 1 region17 continue | 단순 추가 학습으로 병목 해소 여부 확인 | 실패, center precision 병목 판단 | `docs/records/training/20260612_stage1_region17_continue_failure_record.md` |
| 2026-06-13 | Stage 1 seq25 rerun | 128env rerun | region progression 기록 개선 필요 확인 | `docs/records/training/20260613_stage1_seq25_128env_rerun_analysis.md` |
| 2026-06-13 | Stage 1 skip-stalled | 막힌 영역을 skip하고 전체 상태 기록 | `1..14` mastered, `15..25` skipped | `docs/records/training/20260613_stage1_seq25_skipstalled3840_analysis.md` |
| 2026-06-14 | Stage 1 camera audit | 학습 실패와 카메라 실패 분리 | `1..14` operational, `15..25` visible_learning_failed, camera_excluded `0` | `docs/records/training/20260614_stage1_cameraaudit_operating_workspace_analysis.md` |
| 2026-06-14 | Stage 1 16/17 radius ladder | 성공 반경 축소로 precision 한계 확인 | 16/17은 `20mm`까지 mastered, 16의 `15mm` 실패 | `docs/records/training/20260614_stage1_region16_17_radius_ladder_analysis.md` |

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

## Region 16/17 성공 반경 Ladder

| region | success radius | result | stop iteration | training time | best center distance | best top-down XY |
| ---: | ---: | --- | ---: | ---: | ---: | ---: |
| 16 | `35mm` | mastered, `11` successes | `19` | `16.89s` | `34.645mm` | `7.158mm` |
| 17 | `35mm` | mastered, `11` successes | `25` | `23.37s` | `33.052mm` | `23.117mm` |
| 16 | `25mm` | mastered, `10` successes | `134` | `116.11s` | `23.272mm` | `21.975mm` |
| 17 | `25mm` | mastered, `10` successes | `40` | `35.42s` | `23.989mm` | `9.244mm` |
| 16 | `20mm` | mastered, `10` successes | `178` | `154.35s` | `18.443mm` | `18.162mm` |
| 17 | `20mm` | mastered, `10` successes | `82` | `74.98s` | `19.084mm` | `15.563mm` |
| 16 | `15mm` | not mastered, `0` successes | `1200` | `1078.35s` | n/a | n/a |

## 학습 순서 기준

현재 권장 순서는 아래와 같다.

1. Stage 0 workspace-entry/reach-aware smoke로 runtime과 관측 구성을 확인한다.
2. Stage 1 5x5 plane에서 camera-audit와 region_mastery를 같이 기록한다.
3. 실패 영역은 전체 sweep 반복 전에 focus region으로 분리한다.
4. Focus region에서 성공 반경 ladder를 돌린다.
5. `20mm`에서 바로 `15mm`로 닫지 않고 `18mm` 중간 단계를 둔다.
6. `15mm` 목표에는 성공 반경 축소만 쓰지 않고 final center/top-down XY reward를 보강한다.
7. Stage 2 5x5x4 volume은 Stage 1 precision이 안정된 뒤에 연다.

## 해석

Region `15..25`는 카메라 target stereo visibility, camera region match, target estimate error 관점에서 제외할 근거가 없다. 실패는 높은 z 또는 +y 끝 영역에서 final center distance와 top-down XY gate를 동시에 닫지 못하는 문제다.

따라서 실제 운영 workspace는 당분간 `1..14`로 제한하고, region `16/17`은 연구 영역으로 둔다. 성공 반경 후보는 지금은 `20mm`가 현실적이다.
