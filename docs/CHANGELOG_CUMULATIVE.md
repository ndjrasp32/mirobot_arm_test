# MT4 누적 변경 기록

Date: 2026-06-14 KST

이 파일은 코드, 스크립트, 기준값, 문서 구조 변경을 누적해서 찾기 위한 changelog다. 날짜별 상세 근거는 `docs/records/`에 둔다.

## 2026-06-14

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| 문서 구조 | README를 저장소 지도 중심으로 축소 | 처음 온 사람이 최신 기준을 바로 찾게 함 | `README.md` |
| 문서 구조 | `docs/TRAINING_HISTORY.md` 추가 | Stage 0/1 누적 학습 흐름 통합 | `docs/TRAINING_HISTORY.md` |
| 문서 구조 | `docs/ARTIFACT_INDEX.md` 추가 | 영상/그래프/CSV/run directory 위치 통합 | `docs/ARTIFACT_INDEX.md` |
| 문서 구조 | `docs/CHANGELOG_CUMULATIVE.md` 추가 | 코드/스크립트/기준 변경 누적 | 현재 파일 |
| 문서 구조 | `docs/DECISIONS_AND_PROPOSALS.md` 추가 | 사용자 제안, Codex 제안, 최종 결정 통합 | `docs/DECISIONS_AND_PROPOSALS.md` |
| 문서 구조 | `docs/README.md` 추가 | docs 폴더 진입점을 명확히 함 | `docs/README.md` |
| 문서 구조 | `docs/records/`를 단일 평탄 폴더로 정리 | `archive/design/training` 하위 폴더와 중복 인덱스 혼선을 제거 | `docs/README.md` |
| 문서 정리 | 오래된 `20260611_artifact_index.md` 삭제 | 최신 산출물 인덱스가 아니어서 혼선 발생 | `docs/ARTIFACT_INDEX.md`로 대체 |
| 문서 정리 | `region16_17` 상세 기록을 `regions16_20` 기록으로 승격 | 최신 focus ladder가 16..20 전체로 확장됨 | `docs/records/20260614_stage1_regions16_20_radius_ladder_analysis.md` |
| Stage 1 env | `MT4_STAGE1_SUCCESS_RADIUS` 지원 | 성공 반경을 `35/25/20/15mm`처럼 run별로 조정 | `source/mirobot_reach_direct/mirobot_coordinate_curriculum_env.py` |
| Stage 1 env | `MT4_CENTER_SUCCESS_RADIUS`, `MT4_TOP_DOWN_XY_SUCCESS_RADIUS` 유지 | center gate와 top-down XY gate를 따로 override 가능하게 함 | `source/mirobot_reach_direct/mirobot_coordinate_curriculum_env.py` |
| Stage 1 logging | `camera_workspace_audit.csv` 추가 | operational/visible_learning_failed/camera_excluded를 분리 | `docs/records/20260614_stage1_cameraaudit_operating_workspace_analysis.md` |
| Stage 1 logging | `region_mastery.csv`에 best center/top-down XY 정보 보강 | mastered 성공의 품질을 추적 | `docs/records/20260614_stage1_regions16_20_radius_ladder_analysis.md` |
| 문서 기록 | 다음 학습 제안 체크포인트 갱신 | `16`의 15/12mm 병목, `17`의 12mm 병목을 분리 | `docs/DECISIONS_AND_PROPOSALS.md` |
| 학습 기록 | region `16..20` success-radius ladder 결과 반영 | `18..20`은 12mm, `17`은 15mm, `16`은 20mm까지 확인 | `docs/TRAINING_HISTORY.md` |
| 문서 정리 | 변경별 효과 비교표와 흐름 도식 추가 | "무엇을 바꿨더니 무엇이 좋아지고 나빠졌는지"를 한눈에 보이게 함 | `README.md`, `docs/README.md`, `docs/TRAINING_HISTORY.md`, `docs/CURRENT_BASELINE.md`, `docs/DECISIONS_AND_PROPOSALS.md` |

## 2026-06-13

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| Stage 1 curriculum | stalled region skip 구조 추가 | 막힌 영역에서 전체 run이 멈추지 않게 하고 실패 영역을 기록 | `docs/records/20260613_stage1_seq25_skipstalled3840_analysis.md` |
| Stage 1 기록 | `skip_reason`을 `region_mastery.csv`에 남김 | 실패 원인과 진행 상태를 재현 가능하게 함 | `docs/records/20260613_stage1_seq25_skipstalled3840_analysis.md` |
| Stage 1 bugfix | curriculum complete 이후 active region 처리 정리 | 완료 뒤에도 active region이 남는 혼선 방지 | `docs/records/20260613_stage1_seq25_128env_rerun_analysis.md` |

## 2026-06-12

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| Workspace | arm/end center를 로봇팔 쪽으로 10mm 이동 | region 7/9 병목 이후 reachable side 중심 보정 | `docs/records/20260612_stage1_region7_9_focus_axis_analysis.md` |
| Workspace | target center를 tool-tip down offset `35mm` 반영해 낮춤 | 미래 하향 장착 집게 끝점을 기준에 반영 | `docs/records/20260612_stage1_seq25_tipdown35_analysis.md` |
| Stage 1 curriculum | 9영역 순차 mastery 도입 | 영역별 성공을 누적해서 다음 영역으로 진행 | `docs/records/20260612_stage1_seq9_5success_analysis.md` |
| Stage 1 curriculum | 25영역 순차 curriculum 확장 | 전체 plane 가동 가능성 확인 | `docs/records/20260612_stage1_seq25_tipdown35_analysis.md` |

## 2026-06-11

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| Stage 0 reward | phase split 접근 | workspace 진입, top-down 접근, center 접근을 분리 | `docs/records/20260611_phase_split_stage0_analysis.md` |
| Stage 0 gate | top-down height/XY gate 완화 실험 | 관측된 lateral error와 성공 조건 불일치 확인 | `docs/records/20260611_phase_split_relaxed_stage0_analysis.md` |
| Stage 0 observation | reach-aware entry/standoff 계열 | reachable side에서 접근하도록 유도 | `docs/records/20260611_reach_aware_stage0_entrygate_600iter_analysis.md` |
| Stage 1 env | plane center/top-down XY 성공 반경 35mm급 완화 | 초기 plane localization 통과 여부 확인 | `docs/records/20260611_stage1_plane_xy035_center035_analysis.md` |
| Workspace audit | reach-limited workspace 정의 | 실제 MT4 reach 범위 기준 plane/volume curriculum 준비 | `docs/records/20260611_mt4_reach_limited_workspace_audit.md` |

## 2026-06-10

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| Repo 기준 | student coordinate curriculum handoff 정리 | `robotarm_student`와 `robotarm_mt4` 책임 분리 | `docs/records/20260610_student_coordinate_handoff_and_training_plan.md` |
| Stage 0 gate | `workspace_entry_success_radius`를 성공 판정이 아닌 진단 지표로 재정리 | workspace 바깥 근접 성공 latch 방지 | `docs/records/20260610_stage0_gate_fix.md` |

## 2026-05

| 구분 | 변경 | 이유 | 근거 |
| --- | --- | --- | --- |
| Asset | WLKATA/Mirobot URDF 기반 시작 | 실제 MT4 asset fidelity 기준 수립 | `docs/records/20260516_mirobot_urdf_asset_start.md` |
| Mapping | MT4 command-facing 4축 action rule 정의 | 실제 SDK command로 이식 가능한 action만 학습 | `docs/records/20260518_mt4_hardware_transfer_mapping.md` |
| Simulation | dynamic cube target/Mars twin 확인 | contact behavior와 물리 기반 task 준비 | `docs/records/20260518_dynamic_cube_target.md` |
| Archive | 과거 joint sweep/GUI 확인 기록 보존 | 현재 기준 문서와 과거 bring-up 기록을 역할로 분리 | `docs/records/` |
