# MT4 docs map

Date: 2026-06-14 KST

이 폴더는 "현재 기준 -> 누적 기록 -> 산출물 -> 상세 근거" 순서로 읽게 정리한다. 처음 보는 사람은 날짜별 기록부터 열지 말고 아래 순서대로 확인한다.

| 순서 | 파일 | 역할 |
| ---: | --- | --- |
| 1 | `CURRENT_BASELINE.md` | 지금 기준, 최신 학습 결론, 다음 학습 판단 |
| 2 | `TRAINING_HISTORY.md` | Stage 0/1 누적 학습 흐름과 결과 표 |
| 3 | `ARTIFACT_INDEX.md` | 영상, 그래프, CSV, 외부 IsaacLab run directory 위치 |
| 4 | `DECISIONS_AND_PROPOSALS.md` | 사용자 제안, Codex 제안, 결정사항 |
| 5 | `CHANGELOG_CUMULATIVE.md` | 코드/스크립트/문서 기준 변경 누적 |
| 6 | `records/*.md` | 날짜별 상세 근거 기록 |

## 최신 결론

Stage 1 camera-aligned 5x5 plane의 기본 운영 영역은 아직 `1..14`다. `15..25`는 카메라 제외 영역이 아니라 학습/정밀 제어 병목 영역으로 기록한다.

2026-06-14 focus ladder에서 region `16..20`을 성공 반경 `35 -> 25 -> 20 -> 15 -> 12mm`로 줄여 확인했다.

| region | 최신 확인된 최소 성공 반경 | 상태 |
| ---: | ---: | --- |
| 16 | `20mm` | `15mm`, `12mm` 실패 |
| 17 | `15mm` | `12mm` 실패 |
| 18 | `12mm` | 목표 반경 통과 |
| 19 | `12mm` | 목표 반경 통과 |
| 20 | `12mm` | 목표 반경 통과 |

다음 병목은 region `16`의 final center/top-down XY precision이다. `18..20`은 12mm까지 통과했지만 isolated focus 결과이므로, 전체 운영 workspace로 승격하려면 region `16/17` 보강 후 `16..20` 묶음 재검증이 필요하다.

## 변경 효과 요약

처음 보는 사람은 아래 표만 보면 "무엇을 바꿨고, 무엇이 좋아졌고, 무엇이 아직 막혔는지"를 빠르게 볼 수 있다. 자세한 수치와 run directory는 `TRAINING_HISTORY.md`와 `ARTIFACT_INDEX.md`에 둔다.

| 순서 | 변경/실험 | 좋아진 것 | 나빠지거나 남은 것 | 현재 결론 |
| ---: | --- | --- | --- | --- |
| 1 | Stage 0 workspace-entry baseline | task/runtime 포팅 완료 | `inside_workspace_rate=0.0000` | 바로 Stage 1 handoff 불가 |
| 2 | Stage 0 reach-aware entrygate | `inside_workspace_rate=0.4856`, `workspace_entry_success_rate=0.4839` | `center_1cm_rate=0.0000` | entry policy로만 사용 |
| 3 | Stage 1 9-cell sequential | `mastered_region_count=9` | final policy stability 부족 | 25-cell 확장 가능 |
| 4 | Stage 1 25-cell skip-stalled | `14/25` mastered, 실패 영역 기록 완료 | `15..25` skipped, 새 성공 `0` | focus run 필요 |
| 5 | camera audit | `camera_excluded=0/25` | `15..25`는 여전히 `visible_learning_failed` | 카메라보다 precision 문제 |
| 6 | 16..20 radius ladder | `18..20`은 `12mm` 통과 | `16`은 `15/12mm` 실패, `17`은 `12mm` 실패 | `16/17` 보강이 다음 작업 |

```text
Stage 0 runtime
  -> reach-aware entrygate: workspace entry 회복
  -> Stage 1 9-cell: plane localization 확인
  -> Stage 1 25-cell: 1..14 operational, 15..25 병목 분리
  -> camera audit: camera exclusion 아님
  -> 16..20 ladder: 16/17 final precision 병목 확정
```

## 폴더 기준

| 위치 | 용도 |
| --- | --- |
| `records/` | 날짜별 상세 근거 기록. 설계, 학습, 과거 bring-up 기록을 파일명으로 구분 |
| `figures/` | 문서에서 직접 참조하는 작은 도식 |

## 상세 기록 파일명 기준

`records/` 아래는 추가 하위 폴더를 만들지 않는다. 파일명은 `YYYYMMDD_주제_결론.md` 형식으로 두고, 최신 판단은 위의 누적 문서에 먼저 반영한다.

| 찾는 내용 | 대표 파일 |
| --- | --- |
| 최신 16..20 성공 반경 ladder | `records/20260614_stage1_regions16_20_radius_ladder_analysis.md` |
| camera-audit 운영 영역 판단 | `records/20260614_stage1_cameraaudit_operating_workspace_analysis.md` |
| 25영역 skip-stalled sweep | `records/20260613_stage1_seq25_skipstalled3840_analysis.md` |
| region 7/9 접근축 분석 | `records/20260612_stage1_region7_9_focus_axis_analysis.md` |
| camera-aligned 운영 workspace 계획 | `records/20260614_camera_aligned_operating_workspace_plan.md` |
| hardware transfer mapping | `records/20260518_mt4_hardware_transfer_mapping.md` |

원시 launch/stdout/session 로그 전문은 GitHub 문서 기준으로 쓰지 않는다. 필요한 경우 `ARTIFACT_INDEX.md`에 경로만 남긴다.
