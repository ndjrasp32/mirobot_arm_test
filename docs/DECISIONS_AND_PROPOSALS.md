# MT4 대화, 제안, 결정사항 정리

Date: 2026-06-14 KST

이 파일은 사용자 제안, Codex 제안, 최종 결정을 한눈에 보기 위한 기록이다. 날짜별 상세 근거는 `docs/records/`에 둔다.

## 현재 최종 결정

| 주제 | 결정 |
| --- | --- |
| 실제 로봇 motion | Safety Gate 전까지 실행하지 않음 |
| Stage 1 기본 운영 영역 | `1..14`만 operational로 둠 |
| `15..25` 해석 | camera exclusion이 아니라 `visible_learning_failed` |
| region `16..20` 상태 | isolated focus 연구 영역으로 둠 |
| 최종 성공 반경 목표 | `12mm` |
| 최신 focus 결과 | `18..20`은 `12mm`, `17`은 `15mm`, `16`은 `20mm`까지 mastered |
| 다음 개선 | region `16` final center/top-down XY reward 보강 |
| 전체 sweep 재시도 | `16/17`에서 `12mm` 성공 후 진행 |

## 대화와 결정 흐름

| 순서 | 사용자 제안/요청 | Codex 제안/분석 | 최종 결정 |
| ---: | --- | --- | --- |
| 1 | Stage 0 workspace-entry를 MT4에 포팅 | workspace 바깥 근접 성공 latch 위험을 지적 | entry gate를 엄격히 보고 재학습 |
| 2 | Stage 0 결과를 계속 확인 | phase split과 reach-aware 접근 제안 | workspace entry, top-down 접근, center 접근을 분리 |
| 3 | Stage 1 plane으로 넘어감 | 35mm급 완화 성공 반경으로 localization 가능성 확인 제안 | 9영역 순차 curriculum 실행 |
| 4 | 25영역으로 확장 | 막힌 영역에서 전체 run이 멈추지 않도록 skip-stalled 구조 제안 | `1..14` mastered, `15..25` skipped 상태 기록 |
| 5 | 카메라 기준 운영 영역을 정리하고 싶음 | 학습 실패와 카메라 실패를 분리하는 camera audit 제안 | `operational`, `visible_learning_failed`, `camera_excluded` 분류 도입 |
| 6 | 영역 `16-20` 재시도 요청 | 전체 sweep보다 focus region으로 병목을 분리하는 방식 제안 | region `16..20` focus ladder 실행 |
| 7 | 처음엔 `35 -> 30 -> 25`로 줄이는 논의 | 성공 반경과 tool-tip down offset이 섞인 점 확인 | 성공 반경 ladder는 `35, 25, 20, 15, 12`로 정리 |
| 8 | 최종적으로 `12mm` 목표, `16-20` 동일 조건 요청 | 각 region별 최소 통과 반경을 따로 기록해야 한다고 정리 | `18..20`은 `12mm`, `17`은 `15mm`, `16`은 `20mm`까지 확인 |
| 9 | docs 폴더가 다시 엉킨 것 같다고 정리 요청 | 상위 문서가 오래된 16/17 결론에 머문 점 확인 | `docs/README.md` 추가, `regions16_20` 기록으로 승격 |

## 사용자 제안 중 반영된 항목

| 제안 | 반영 상태 |
| --- | --- |
| 영역별로 다시 돌려 병목을 확인 | region `16..20` focus run으로 반영 |
| 성공 반경을 큰 값에서 작은 값으로 줄이기 | `35mm -> 25mm -> 20mm -> 15mm -> 12mm` ladder 실행 |
| 최종 목표를 `12mm`로 두기 | region `18..20`에서 `12mm`까지 확인 |
| 매 메시지마다 진행률 공유 | 학습/정리 작업 중 단계와 예상 시간을 업데이트하는 운영 방식으로 반영 |
| 이전 개선 사항, 순서, 방법 정리 | `docs/TRAINING_HISTORY.md`, `docs/CHANGELOG_CUMULATIVE.md`로 통합 |
| 영상과 그래프를 찾기 쉽게 정리 | `docs/ARTIFACT_INDEX.md`로 통합 |
| GitHub 처음 오는 사람이 파일명만 보고 찾게 정리 | README, `docs/README.md`, `docs/`의 명확한 파일명으로 재구성 |

## Codex 제안 중 채택된 항목

| 제안 | 이유 | 상태 |
| --- | --- | --- |
| Safety Gate 전 실제 robot motion 보류 | hardware risk 관리 | 채택 |
| camera audit로 실패 원인 분리 | perception 문제와 control/reward 문제를 구분 | 채택 |
| stalled region skip | 전체 run이 막힌 영역에서 정지하지 않게 함 | 채택 |
| region focus run | 전체 25영역 반복보다 병목 분리 효율이 높음 | 채택 |
| 성공 반경 ladder | precision 한계를 단계별로 분리 | 채택 |
| final precision reward 보강 | `16/17`의 12mm 병목 보완 | 다음 작업 |

## 2026-06-14 다음 학습 제안 체크포인트

| 우선순위 | 제안 | 판단 기준 |
| ---: | --- | --- |
| 1 | region `16`의 `15mm` 실패 로그를 center/top-down XY로 분해 | `20mm`는 되지만 `15mm`가 안 되는 직접 원인 확인 |
| 2 | final 25mm 이내 center/top-down XY reward를 보강 | 성공 반경 축소만으로는 region `16`이 닫히지 않는 문제 보완 |
| 3 | region `16`을 `15mm -> 12mm` 순서로 재시도 | `15mm` 10회 성공 후 `12mm` 진입 |
| 4 | region `17`은 보강 조건으로 `12mm`만 재시도 | 이미 `15mm`는 통과했으므로 12mm 병목만 확인 |
| 5 | `16/17`이 `12mm`를 통과한 뒤 `16..20` 묶음 재검증 | isolated success를 운영 후보로 승격할지 판단 |
| 6 | Safety Gate 전 실제 MT4 motion은 계속 보류 | sim policy가 안정돼도 hardware risk는 별도 검증 필요 |

## 보류된 항목

| 항목 | 보류 이유 |
| --- | --- |
| 실제 MT4 motion | Safety Gate 미완료 |
| Stage 2 volume 본격 학습 | Stage 1 precision이 아직 안정되지 않음 |
| `15..25` 전체 운영 승격 | region `16/17`이 아직 `12mm`를 통과하지 못함 |
| 전체 `15..25` 재시도 | focus region에서 실패 원인을 먼저 줄여야 함 |
