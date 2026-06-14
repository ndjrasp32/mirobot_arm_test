# 2026-06-14 MT4 카메라 정렬 기반 운영 가동 영역 계획

## 목적

Stage 1 5x5 plane 학습은 지금까지 내부 좌표로 정한 reach-limited workspace를 카메라 투영과 맞추며 진행했다. 2026-06-13 sweep 결과에서 `1..14` 영역은 mastered, `15..25` 영역은 skip됐으므로, 다음 실험은 전체 workspace를 억지로 모두 성공시키는 방향이 아니라 카메라 기준으로 실제 운영 가능한 영역을 먼저 확정하는 방향으로 전환한다.

이 기록은 실제 MT4 motion을 실행하기 전, simulation 안에서 카메라와 학습 결과를 함께 써서 "운영 가능 영역"과 "보이지만 제어/접근이 실패한 영역"을 분리하는 계획이다.

## 결론

다음 기준을 채택한다.

1. Stage 1 plane curriculum은 계속 `5x5` 카메라 영역으로 순차 진행한다.
2. 오래 실패한 영역은 전체 학습을 멈추지 않고 skip 처리해 다음 영역으로 넘어간다.
3. 각 영역은 학습 성공 여부만으로 판단하지 않고, 카메라 관측 안정성도 같이 기록한다.
4. 운영 가능 영역은 `mastered`이면서 body stereo camera에서 target region 추정이 안정적인 영역으로 본다.
5. `skipped`이지만 카메라 추정이 안정적인 영역은 "카메라 문제가 아니라 접근/제어/reward 병목"으로 분류한다.
6. 실제 MT4 motion은 기존 Safety Gate를 통과하기 전까지 실행 기준으로 올리지 않는다.

## 기준 값 또는 변경 내용

현재 workspace 기준은 유지한다.

| 항목 | 값 |
| --- | --- |
| arm/end workspace center | `(-0.068, 0.000, 0.103)` |
| target workspace center | `(-0.068, 0.000, 0.068)` |
| workspace size | `(0.045, 0.095, 0.055)` |
| Stage 1 plane | x=`-0.068`, y/z `5x5` |
| tool-tip down offset | `0.035 m` |
| center success radius | `0.012 m` |
| top-down XY success radius | `0.012 m` |
| mastery target | `10` new successes per region |
| stalled-region policy | no new success for configured env steps -> skipped and continue |

새로 기록할 영역별 camera workspace audit 기준:

| 항목 | 기본 판단 |
| --- | --- |
| `target_stereo_visible_rate` | body left/right camera가 target을 안정적으로 보는지 확인 |
| `camera_region_match_rate` | stereo 추정 region이 내부 target region과 맞는지 확인 |
| `mean_target_estimate_error` | 카메라 추정 target 좌표와 내부 target 좌표의 평균 오차 |
| `success_count/mastered/skipped` | 해당 영역의 실제 학습 성공/정체 상태 |
| `operating_status` | `operational`, `visible_learning_failed`, `camera_excluded`, `pending` 중 하나 |

기본 분류 기준은 다음과 같다.

| status | 의미 |
| --- | --- |
| `operational` | 카메라 추정이 안정적이고, 학습도 mastered |
| `visible_learning_failed` | 카메라 추정은 안정적인데 영역이 skipped 또는 미성공 |
| `camera_excluded` | body stereo visibility, region match, target estimate 중 하나가 기준 미달 |
| `pending` | 아직 충분한 샘플이나 학습 판단이 쌓이지 않음 |

## 검증 방법

1. `camera_workspace_audit.csv`가 run directory에 생성되는지 확인한다.
2. CSV에 25개 영역별 sample count, camera visibility, region match, estimate error, mastery 상태가 기록되는지 확인한다.
3. `region_mastery.csv`와 `camera_workspace_audit.csv`를 같이 보고 운영 영역을 확정한다.
4. 짧은 smoke run에서 CSV 생성만 확인한 뒤, 본 학습 run을 실행한다.
5. 본 학습 결과는 `docs/records/training/`에 별도 dated record로 남긴다.

## 다음 작업

1. `Mirobot-Coordinate-Plane-Direct-v0` 환경에 영역별 camera workspace audit 누적 기록을 추가한다.
2. sweep script에서 run name에 camera audit 성격이 드러나게 한다.
3. 먼저 짧은 실행으로 CSV 생성과 stop/skip 동작을 확인한다.
4. 이후 25영역 sweep을 다시 실행해 운영 가능 영역을 `operational` 상태로 확정한다.
5. `visible_learning_failed` 영역은 전체 sweep을 늘리기보다 focus run으로 reward/접근축 문제를 따로 본다.

## English Notes

The next MT4 Stage 1 experiment should use camera-aligned curriculum data as an operating-workspace audit. A region is not considered operational just because it is sampled; it should be both camera-stable and mastered. Regions that are camera-stable but skipped are treated as reach/control/reward bottlenecks rather than perception failures.
