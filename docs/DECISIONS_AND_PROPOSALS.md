# MT4 대화, 제안, 결정사항 정리

Date: 2026-06-14 KST

이 파일은 사용자 제안, Codex 제안, 최종 결정을 한눈에 보기 위한 기록이다. 날짜별 상세 근거는 `docs/records/`에 둔다.

## 현재 최종 결정

| 주제 | 결정 |
| --- | --- |
| 실제 로봇 motion | Safety Gate 전까지 실행하지 않음 |
| Stage 1 운영 영역 | `1..14`만 operational로 둠 |
| `15..25` 해석 | camera exclusion이 아니라 `visible_learning_failed` |
| region `16/17` 상태 | `20mm`까지 학습 가능 확인, 운영 후보는 아직 연구 상태 |
| 다음 반경 | `15mm` 직행 반복 대신 `18mm` 중간 단계 |
| 다음 개선 | final 25mm 이내 center/top-down XY reward 보강 |
| 전체 sweep 재시도 | `16/17`에서 `15mm` 성공 후 진행 |

## 대화와 결정 흐름

| 순서 | 사용자 제안/요청 | Codex 제안/분석 | 최종 결정 |
| ---: | --- | --- | --- |
| 1 | Stage 0 workspace-entry를 MT4에 포팅 | workspace 바깥 근접 성공 latch 위험을 지적 | entry gate를 엄격히 보고 재학습 |
| 2 | Stage 0 결과를 계속 확인 | phase split과 reach-aware 접근 제안 | workspace entry, top-down 접근, center 접근을 분리 |
| 3 | Stage 1 plane으로 넘어감 | 35mm급 완화 성공 반경으로 localization 가능성 확인 제안 | 9영역 순차 curriculum 실행 |
| 4 | 25영역으로 확장 | 막힌 영역에서 전체 run이 멈추지 않도록 skip-stalled 구조 제안 | `1..14` mastered, `15..25` skipped 상태 기록 |
| 5 | 카메라 기준 운영 영역을 정리하고 싶음 | 학습 실패와 카메라 실패를 분리하는 camera audit 제안 | `operational`, `visible_learning_failed`, `camera_excluded` 분류 도입 |
| 6 | 영역 `16-20` 재시도 요청 | 전체 sweep보다 focus region으로 병목을 분리하는 방식 제안 | region `16`, `17`부터 focus run |
| 7 | 처음엔 `35 -> 30 -> 25`로 줄이는 논의 | 성공 반경과 tool-tip down offset이 섞인 점 확인 | 성공 반경 ladder는 `35, 25, 20, 15, 12`로 정리 |
| 8 | 다시 `35, 25, 20, 15, 12`로 확정 | `15mm`에서 실패하면 `12mm` 직행보다 중간 단계/보상 조정 필요 제안 | `35 -> 25 -> 20`까지 확인, `15` 실패 후 `18` 중간 단계로 수정 |
| 9 | 학습 결과와 진행상황을 GitHub에 정리 요청 | 날짜별 md가 흩어져 있어 통합 진입점 필요 분석 | README, CURRENT_BASELINE, TRAINING_HISTORY, ARTIFACT_INDEX, CHANGELOG, DECISIONS로 재구성 |

## 사용자 제안 중 반영된 항목

| 제안 | 반영 상태 |
| --- | --- |
| 영역별로 다시 돌려 병목을 확인 | region `16`, `17` focus run으로 반영 |
| 성공 반경을 큰 값에서 작은 값으로 줄이기 | `35mm -> 25mm -> 20mm -> 15mm` ladder 실행 |
| 매 메시지마다 진행률 공유 | 학습/정리 작업 중 단계와 예상 시간을 업데이트하는 운영 방식으로 반영 |
| 이전 개선 사항, 순서, 방법 정리 | `docs/TRAINING_HISTORY.md`, `docs/CHANGELOG_CUMULATIVE.md`로 통합 |
| 영상과 그래프를 찾기 쉽게 정리 | `docs/ARTIFACT_INDEX.md`로 통합 |
| GitHub 처음 오는 사람이 파일명만 보고 찾게 정리 | README와 `docs/`의 명확한 파일명으로 재구성 |

## Codex 제안 중 채택된 항목

| 제안 | 이유 | 상태 |
| --- | --- | --- |
| Safety Gate 전 실제 robot motion 보류 | hardware risk 관리 | 채택 |
| camera audit로 실패 원인 분리 | perception 문제와 control/reward 문제를 구분 | 채택 |
| stalled region skip | 전체 run이 막힌 영역에서 정지하지 않게 함 | 채택 |
| region focus run | 전체 25영역 반복보다 병목 분리 효율이 높음 | 채택 |
| `15mm` 실패 후 `18mm` 중간 단계 | `20mm`와 `15mm` 사이 난도 차이를 완충 | 채택 |
| final precision reward 보강 | 성공 반경만 줄이는 방식의 한계 보완 | 다음 작업 |

## 보류된 항목

| 항목 | 보류 이유 |
| --- | --- |
| 실제 MT4 motion | Safety Gate 미완료 |
| Stage 2 volume 본격 학습 | Stage 1 precision이 아직 안정되지 않음 |
| `12mm` 성공 반경 | `15mm`도 실패했으므로 현재는 너무 이른 목표 |
| 전체 `15..25` 재시도 | focus region에서 15mm 성공 조건을 먼저 만들어야 함 |
