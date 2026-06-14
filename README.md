# robotarm_mt4

WLKATA Mirobot/MT4 asset, hardware-transfer mapping, IsaacLab direct-RL training, and safe-simulation baseline repository.

이 저장소를 처음 열면 아래 순서로 보면 된다.

| 순서 | 파일 | 용도 |
| ---: | --- | --- |
| 1 | `docs/README.md` | docs 폴더 지도와 최신 결론 요약 |
| 2 | `docs/CURRENT_BASELINE.md` | 지금 기준, 최신 결과, 바로 다음 학습 판단 |
| 3 | `docs/TRAINING_HISTORY.md` | Stage 0/1 누적 학습 기록과 결과 요약 |
| 4 | `docs/ARTIFACT_INDEX.md` | 영상, 그래프, CSV, 외부 run directory 위치 |
| 5 | `docs/DECISIONS_AND_PROPOSALS.md` | 사용자 제안, Codex 제안, 최종 결정사항 |
| 6 | `docs/CHANGELOG_CUMULATIVE.md` | 코드/스크립트/환경 기준 변경 누적 기록 |

## 현재 결론

2026-06-14 KST 기준 실제 MT4 motion은 아직 실행하지 않았다. 모든 최근 결과는 IsaacLab headless simulation 기준이다.

Stage 1 camera-aligned 5x5 plane에서는 `1..14`가 기본 운영 가능 영역이고, `15..25`는 카메라 문제가 아니라 중심 접근/정밀 제어 병목으로 분류한다. Focus ladder 결과 region `18`, `19`, `20`은 성공 반경 `12mm`까지 mastered됐고, region `17`은 `15mm`까지, region `16`은 `20mm`까지 mastered됐다.

다음 학습 판단은 region `16`의 `15mm/12mm` 병목을 먼저 줄이고, region `17`의 `12mm`를 재시도한 뒤 `16..20` 묶음 재검증으로 운영 승격 여부를 판단하는 것이다. 실제 로봇 구동은 `docs/CURRENT_BASELINE.md`의 Safety Gate를 통과한 뒤에만 다룬다.

## 한눈에 보는 개선 효과

| 바꾼 것 | 좋아진 점 | 남은 문제 | 다음 판단 |
| --- | --- | --- | --- |
| Stage 0 gate/reach-aware 재정의 | `inside_workspace_rate`가 초기 `0.0000`에서 entrygate 기준 `0.4856`까지 올라감 | `center_1cm_rate=0.0000`이라 최종 grasp policy는 아님 | Stage 1 bootstrap용으로만 사용 |
| Stage 1 sequential region mastery | 9-cell plane에서 `mastered_region_count=9` 달성 | final checkpoint 안정성은 부족 | 25-cell 확장으로 진행 |
| 25-cell skip-stalled 기록 | 막힌 곳에서 멈추지 않고 `14/25` mastered, `11/25` skipped로 상태가 남음 | `15..25`는 12mm 조건에서 새 성공 `0` | 실패 영역 focus run으로 분리 |
| camera audit | `camera_excluded=0/25`, `15..25=visible_learning_failed`로 원인 분리 | 운영 영역은 아직 `1..14` | perception보다 precision 보강 우선 |
| 16..20 success-radius ladder | `18..20`은 `12mm`, `17`은 `15mm`, `16`은 `20mm`까지 통과 | `16`의 `15/12mm`, `17`의 `12mm` 실패 | final center/top-down XY reward 보강 |

## 저장소 역할

이 저장소에서 관리한다.

- MT4/Mirobot URDF/USD asset 기준
- Isaac joint/action to MT4 hardware command mapping
- camera-aligned operating workspace와 perception baseline
- MT4 coordinate curriculum 학습 기록과 산출물
- 실제 로봇 motion 전 safety gate

이 저장소에서 관리하지 않는다.

- 학생용 장기 curriculum archive
- 실제 로봇 무검증 motion 실행
- 원시 stdout/launch 로그 전체 보관

## 핵심 Task

| 구분 | 값 |
| --- | --- |
| Python package | `source/mirobot_reach_direct` |
| Reach task | `Mirobot-Reach-Pregrasp-Direct-v0` |
| Coordinate Stage 1 task | `Mirobot-Coordinate-Plane-Direct-v0` |
| Coordinate Stage 2 task | `Mirobot-Coordinate-Volume-Direct-v0` |
| Mars twin tasks | `Mirobot-Mars-Twin-{Pick,Place,Stack,Push,Pull}-Direct-v0` |

## 반복 실행 명령

시각 확인:

```bash
./scripts/inspect_mirobot_asset.sh
./scripts/check_mirobot_joint_limits.sh
./scripts/show_mt4_hardware_mapping_gui.sh
./scripts/view_mirobot_mars_twin_gui.sh --mission push
```

학습:

```bash
./scripts/train_mirobot_coordinate_stage0_workspace_entry_128_300.sh
./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
./scripts/train_mirobot_coordinate_stage1_plane_sweep25_skip.sh
./scripts/train_mirobot_coordinate_stage2_volume_128_600.sh
```

Stage 1 성공 반경 ladder 예:

```bash
MT4_STAGE1_SUCCESS_RADIUS=0.035 ./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
MT4_STAGE1_SUCCESS_RADIUS=0.025 ./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
MT4_STAGE1_SUCCESS_RADIUS=0.020 ./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
MT4_STAGE1_SUCCESS_RADIUS=0.015 ./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
MT4_STAGE1_SUCCESS_RADIUS=0.012 ./scripts/train_mirobot_coordinate_stage1_plane_128_600.sh
```

분석:

```bash
./scripts/plot_and_select_mirobot_best.sh
./scripts/play_mirobot_best.sh
```

## 문서 운영 규칙

새로 온 사람은 `docs/CURRENT_BASELINE.md`와 `docs/TRAINING_HISTORY.md`만 읽어도 현재 상태를 이해할 수 있어야 한다. 날짜별 상세 md는 `docs/records/*.md`에 평탄하게 보관하며, 최신 판단을 직접 찾는 시작점으로 쓰지 않는다.

원시 로그는 `training_logs/run_stdout/`, `training_logs/session_logs/`, `training_logs/launch/`, `logs/` 아래에 남을 수 있지만 `.gitignore` 대상이다. GitHub에는 요약 문서, 대표 그래프, 대표 영상, CSV만 올린다.
