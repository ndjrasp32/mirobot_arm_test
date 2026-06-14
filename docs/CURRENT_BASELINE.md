# 현재 기준 - robotarm_mt4

Date: 2026-06-14 KST

## 한 줄 요약

MT4 Stage 1은 camera-aligned 5x5 plane 기준 `1..14`가 운영 가능 영역이고, region `16`, `17`은 성공 반경 `20mm`까지 학습 가능하다. `15mm`는 아직 실패했으므로 다음 목표는 `18mm -> 15mm` ladder와 final precision reward 보강이다.

## 오늘 보는 순서

| 순서 | 파일 | 이유 |
| ---: | --- | --- |
| 1 | `README.md` | 저장소 전체 지도 |
| 2 | `docs/CURRENT_BASELINE.md` | 현재 기준 |
| 3 | `docs/TRAINING_HISTORY.md` | 누적 학습 흐름 |
| 4 | `docs/ARTIFACT_INDEX.md` | 영상/그래프/CSV 위치 |
| 5 | `docs/DECISIONS_AND_PROPOSALS.md` | 사용자 제안, Codex 제안, 결정사항 |
| 6 | `docs/records/README.md` | 날짜별 상세 근거 |

## 현재 작업 범위

이 저장소는 실제 MT4 이식에 가까운 기준만 관리한다.

| 포함 | 제외 |
| --- | --- |
| Mirobot/MT4 URDF/USD asset check | 학생용 장기 curriculum archive |
| Isaac action to MT4 command mapping | 검증 없는 실제 robot motion |
| camera/perception baseline | 원시 stdout/launch 로그 전체 Git 보관 |
| coordinate curriculum 학습 결과 | 임시 실험 메모의 README 승격 |
| Safety Gate | hardware safety gate 이전의 자동 구동 |

## 최신 학습 상태

| 항목 | 현재값 |
| --- | --- |
| Stage 0 | workspace-entry/reach-aware 계열로 task/runtime 포팅 확인, 안정 policy handoff는 미완 |
| Stage 1 기본 영역 | camera-audit sweep에서 `1..14` mastered/operational |
| Stage 1 실패 영역 | `15..25`는 `visible_learning_failed`, camera exclusion은 아님 |
| Region 16/17 focus | `35mm -> 25mm -> 20mm` mastered |
| Region 16 precision probe | `15mm` 실패, `1200` iteration 동안 success `0` |
| 운영 성공 반경 후보 | 당장은 `20mm` |
| 다음 성공 반경 | `18mm` 중간 단계 후 `15mm` 재시도 |

상세 누적 표는 `docs/TRAINING_HISTORY.md`에 둔다.

## 현재 Workspace

| 항목 | 값 |
| --- | --- |
| arm/end center | `(-0.068, 0.000, 0.103)` |
| target center, tool-tip down offset 반영 | `(-0.068, 0.000, 0.068)` |
| target workspace size | `(0.045, 0.095, 0.055)` |
| target min | `(-0.0905, -0.0475, 0.0405)` |
| target max | `(-0.0455, 0.0475, 0.0955)` |
| Stage 1 plane | 5x5, x=`-0.0680` |
| Stage 1 cell size | y/z `(0.0190, 0.0110)` |
| Stage 2 volume | 5x5x4 |
| Stage 2 cell size | x/y/z `(0.0090, 0.0190, 0.0138)` |

## Hardware Transfer Rule

| MT4 command | Isaac joint |
| --- | --- |
| X | `joint_1` |
| Y | `joint_2_1` |
| Z | `joint_3` |
| A | `gripper_body_joint` |

`joint_2_2`, `joint_4`, `joint_l4`는 URDF/USD에는 남겨두지만 policy action으로 쓰지 않는다. 현재 내부 target은 `joint_2_2 = joint_2_1`, `joint_4 = 0.65`, `joint_l4 = 0.35`다.

## Perception Baseline

| 카메라 | 역할 |
| --- | --- |
| body/front camera | 작업 공간 전체와 target 관찰 |
| wrist/downward camera | grasp 직전 상대 위치, 높이, 접촉 후보 확인 |

전환 순서는 내부 target 좌표 baseline, camera-estimated target 좌표, 필요 시 image feature 포함 순서다. 현재 region `15..25` 실패는 카메라 가시성 실패가 아니라 제어/reward precision 병목으로 기록한다.

## Safety Gate

실제 MT4 motion은 아래 항목이 기록되기 전까지 실행 기준으로 올리지 않는다.

- home pose joint table
- conservative joint limits
- Isaac joint/action to MT4 SDK command mapping
- no-motion connection check
- low-speed single-joint check
- emergency stop and recovery procedure

## 다음 작업

1. Region `16`에서 성공 반경 `18mm`를 먼저 확인한다.
2. `18mm`가 mastered되면 final 25mm 이내 center/top-down XY reward를 보강하고 `15mm`를 재시도한다.
3. Region `16`에서 통과한 조건을 region `17`에 복제한다.
4. `16/17`의 `15mm` 성공 후에만 `15..25` 전체 sweep을 다시 연다.
5. 실제 robot motion은 Safety Gate 완료 전까지 보류한다.
