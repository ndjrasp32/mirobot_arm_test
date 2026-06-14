# MT4 날짜별 상세 기록 인덱스

이 폴더는 근거 보관용이다. 최신 판단은 먼저 아래 파일에서 확인한다.

| 우선순위 | 파일 |
| ---: | --- |
| 1 | `../../README.md` |
| 2 | `../CURRENT_BASELINE.md` |
| 3 | `../TRAINING_HISTORY.md` |
| 4 | `../ARTIFACT_INDEX.md` |
| 5 | `../CHANGELOG_CUMULATIVE.md` |
| 6 | `../DECISIONS_AND_PROPOSALS.md` |

## 폴더 기준

| 위치 | 용도 |
| --- | --- |
| `design/` | asset, hardware mapping, perception, workspace, safety처럼 기준이 되는 설계 기록 |
| `training/` | 학습 실행, 영상, 수치, 분석, 다음 판단 기록 |
| `archive/` | 현재 기준에는 직접 쓰지 않지만 보존할 과거 GUI 확인, joint sweep, bring-up 메모 |

## 핵심 설계 기록

| 주제 | 문서 |
| --- | --- |
| URDF asset 시작 | `design/20260516_mirobot_urdf_asset_start.md` |
| 공식 MT4 URDF 확인 | `design/20260518_official_mt4_urdf_check.md` |
| hardware transfer mapping | `design/20260518_mt4_hardware_transfer_mapping.md` |
| dynamic cube target | `design/20260518_dynamic_cube_target.md` |
| dual Pi camera perception | `design/20260608_dual_pi_camera_perception_plan.md` |
| student coordinate handoff | `design/20260610_student_coordinate_handoff_and_training_plan.md` |
| reach-limited workspace audit | `design/20260611_mt4_reach_limited_workspace_audit.md` |
| camera-aligned operating workspace | `design/20260614_camera_aligned_operating_workspace_plan.md` |

## 핵심 학습 기록

| 날짜 | 단계 | 문서 | 결론 |
| --- | --- | --- | --- |
| 2026-06-10 | Stage 0 | `training/20260610_stage0_workspace_entry_result.md` | task/runtime 포팅 확인, workspace 진입은 실패 |
| 2026-06-10 | Stage 0 | `training/20260610_stage0_gate_fix.md` | entry gate 기준 조정 결정 |
| 2026-06-10 | Stage 0 | `training/20260610_stage0_workspace_entry_retrain.md` | stricter gate 재학습, 안정 성공 미완 |
| 2026-06-11 | Stage 0 | `training/20260611_phase_split_stage0_analysis.md` | phase split 1차 분석 |
| 2026-06-11 | Stage 0 | `training/20260611_phase_split_relaxed_stage0_analysis.md` | gate 완화 분석 |
| 2026-06-11 | Stage 0 | `training/20260611_reach_aware_stage0_post_latch_analysis.md` | post-latch 관찰 |
| 2026-06-11 | Stage 0 | `training/20260611_reach_aware_stage0_standoff2cm_600iter_analysis.md` | standoff 2cm 분석 |
| 2026-06-11 | Stage 0 | `training/20260611_reach_aware_stage0_entrygate_600iter_analysis.md` | 최신 Stage 0 근거 |
| 2026-06-11 | Stage 1 | `training/20260611_stage1_plane_xy035_center035_analysis.md` | 35mm 완화 plane 일부 성공 |
| 2026-06-12 | Stage 1 | `training/20260612_stage1_seq9_5success_analysis.md` | 9영역 순차 mastery 성공 |
| 2026-06-12 | Stage 1 | `training/20260612_stage1_seq25_tipdown35_analysis.md` | 25영역에서 16 이후 병목 |
| 2026-06-12 | Stage 1 | `training/20260612_stage1_region7_9_focus_axis_analysis.md` | 7/9 접근축 분석 |
| 2026-06-12 | Stage 1 | `training/20260612_stage1_region17_continue_failure_record.md` | region 17 단순 추가 학습 실패 |
| 2026-06-13 | Stage 1 | `training/20260613_stage1_seq25_128env_rerun_analysis.md` | 128env rerun 결과 |
| 2026-06-13 | Stage 1 | `training/20260613_stage1_seq25_skipstalled3840_analysis.md` | stalled region skip 구조로 1..14 mastered |
| 2026-06-14 | Stage 1 | `training/20260614_stage1_cameraaudit_operating_workspace_analysis.md` | 1..14 operational, 15..25 visible_learning_failed |
| 2026-06-14 | Stage 1 focus | `training/20260614_stage1_region16_17_radius_ladder_analysis.md` | 16/17은 20mm까지 mastered, 16의 15mm 실패 |

## 기록 양식

학습 기록은 아래 순서를 기본값으로 쓴다.

1. 목적
2. 코드/스크립트 변경
3. 학습 실행
4. 영상
5. 최종 지표
6. 분석
7. 다음 판단
8. English Notes, 필요할 때만

설계 기록은 아래 순서를 기본값으로 쓴다.

1. 목적
2. 결론
3. 기준 값 또는 변경 내용
4. 검증 방법
5. 다음 작업
6. English Notes, 필요할 때만

## 삭제/보존 기준

중복되는 “최신 인덱스” 문서는 `docs/`의 통합 인덱스로 대체한다. 원시 로그 전문은 GitHub에 올리지 않고, 날짜별 기록에는 run directory, 대표 영상, 대표 그래프, CSV 경로만 남긴다.
