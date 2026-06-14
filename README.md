# robotarm_mt4

WLKATA Mirobot/MT4 asset, hardware-transfer mapping, IsaacLab direct-RL training, and safe-simulation baseline repository.

이 저장소를 처음 열면 아래 순서로 보면 된다.

| 순서 | 파일 | 용도 |
| ---: | --- | --- |
| 1 | `docs/CURRENT_BASELINE.md` | 지금 기준, 최신 결과, 바로 다음 학습 판단 |
| 2 | `docs/TRAINING_HISTORY.md` | Stage 0/1 누적 학습 기록과 결과 요약 |
| 3 | `docs/ARTIFACT_INDEX.md` | 영상, 그래프, CSV, 외부 run directory 위치 |
| 4 | `docs/CHANGELOG_CUMULATIVE.md` | 코드/스크립트/환경 기준 변경 누적 기록 |
| 5 | `docs/DECISIONS_AND_PROPOSALS.md` | 사용자 제안, Codex 제안, 최종 결정사항 |
| 6 | `docs/records/README.md` | 날짜별 상세 근거 기록 인덱스 |

## 현재 결론

2026-06-14 KST 기준 실제 MT4 motion은 아직 실행하지 않았다. 모든 최근 결과는 IsaacLab headless simulation 기준이다.

Stage 1 camera-aligned 5x5 plane에서는 `1..14`가 운영 가능 영역이고, `15..25`는 카메라 문제가 아니라 중심 접근/정밀 제어 병목으로 분류한다. Focus run 결과 region `16`, `17`은 성공 반경 `20mm`까지 mastered됐고, region `16`의 `15mm`는 `1200` iteration 동안 성공 0회로 실패했다.

다음 학습 판단은 `15mm` 직행 반복보다 `18mm -> 15mm` 중간 단계와 final precision reward 보강이다. 실제 로봇 구동은 `docs/CURRENT_BASELINE.md`의 Safety Gate를 통과한 뒤에만 다룬다.

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
```

분석:

```bash
./scripts/plot_and_select_mirobot_best.sh
./scripts/play_mirobot_best.sh
```

## 문서 운영 규칙

새로 온 사람은 `docs/CURRENT_BASELINE.md`와 `docs/TRAINING_HISTORY.md`만 읽어도 현재 상태를 이해할 수 있어야 한다. 날짜별 상세 md는 근거 보관용이며, 최신 판단을 직접 찾는 시작점으로 쓰지 않는다.

원시 로그는 `training_logs/run_stdout/`, `training_logs/session_logs/`, `training_logs/launch/`, `logs/` 아래에 남을 수 있지만 `.gitignore` 대상이다. GitHub에는 요약 문서, 대표 그래프, 대표 영상, CSV만 올린다.
