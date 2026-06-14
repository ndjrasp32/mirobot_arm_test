# MT4 학습 산출물 인덱스

Date: 2026-06-14 KST

이 파일은 영상, 그래프, CSV, 외부 IsaacLab run directory를 찾기 위한 통합 인덱스다. 원시 stdout/launch 로그 전문은 `.gitignore` 대상이며 GitHub에는 대표 산출물과 요약 기록만 올린다.

## Git에 보관하는 산출물

| 위치 | 내용 | 비고 |
| --- | --- | --- |
| `training_logs/videos/` | 대표 학습/재생 mp4 | Stage 0과 초기 Stage 1 대표 영상 |
| `training_logs/figures/` | Stage 0 그래프 png | 2026-06-10 workspace-entry 계열 |
| `logs/plots/20260612_seq9_5success/` | Stage 1 seq9 그래프/CSV | reward, success mastery, region progress |
| `logs/plots/*.png` | checkpoint selection plot | reach/pregrasp 계열 plotting 산출물 |
| `logs/plots/*.csv` | checkpoint summary | best checkpoint 판단용 |
| `logs/video_previews/` | contact preview jpg | GUI/영상 preview |

## Git에 올리지 않는 산출물

| 위치 | 이유 |
| --- | --- |
| `training_logs/launch/` | 원시 launch log가 크고 반복 생성됨 |
| `training_logs/run_stdout/` | stdout 전문이라 요약 문서로 대체 |
| `training_logs/session_logs/` | tmux/session 로그 전문이라 요약 문서로 대체 |
| `logs/run_output/` | 실행 stdout 전문이라 요약 문서로 대체 |
| 외부 IsaacLab run directory | checkpoint/event/raw CSV가 크므로 경로와 요약만 기록 |

`.gitignore`가 위 원시 로그 위치를 제외한다. 필요한 경우 로컬에서는 남겨두고, GitHub에는 요약 md와 대표 그래프/영상만 남긴다.

## 최신 주요 Run Directory

| 날짜 | run | 위치 | 요약 |
| --- | --- | --- | --- |
| 2026-06-14 | Stage 1 camera-audit seq25 | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_12-37-16_mt4_coordinate_plane_seq25_cameraaudit_skipstalled3840_tipdown35mm_128env_3000iter` | `1..14` operational, `15..25` visible_learning_failed |
| 2026-06-14 | region 16 success35mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-33-49_mt4_coordinate_plane_seq25_10success_success35mm_tipdown35mm_region16_128env_1200iter` | mastered |
| 2026-06-14 | region 17 success35mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-34-41_mt4_coordinate_plane_seq25_10success_success35mm_tipdown35mm_region17_128env_1200iter` | mastered |
| 2026-06-14 | region 16 success25mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-35-11_mt4_coordinate_plane_seq25_10success_success25mm_tipdown35mm_region16_128env_1200iter` | mastered |
| 2026-06-14 | region 17 success25mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-37-15_mt4_coordinate_plane_seq25_10success_success25mm_tipdown35mm_region17_128env_1200iter` | mastered |
| 2026-06-14 | region 16 success20mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-37-58_mt4_coordinate_plane_seq25_10success_success20mm_tipdown35mm_region16_128env_1200iter` | mastered |
| 2026-06-14 | region 17 success20mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-40-40_mt4_coordinate_plane_seq25_10success_success20mm_tipdown35mm_region17_128env_1200iter` | mastered |
| 2026-06-14 | region 16 success15mm | `/home/spark-robotics/work/isaac/src/IsaacLab/logs/rsl_rl/mirobot_coordinate_curriculum_direct/2026-06-14_18-42-03_mt4_coordinate_plane_seq25_10success_success15mm_tipdown35mm_region16_128env_1200iter` | not mastered |

## 대표 영상

| 단계 | 위치 |
| --- | --- |
| Stage 0 phase split | `training_logs/videos/2026-06-11_14-34-18_mt4_phase_split_stage0_128env_300iter_video/` |
| Stage 0 phase split progress | `training_logs/videos/2026-06-11_14-37-09_mt4_phase_split_progress_stage0_128env_300iter_video/` |
| Stage 0 relaxed | `training_logs/videos/2026-06-11_14-45-39_mt4_phase_split_relaxed_stage0_128env_300iter_video/` |
| Stage 0 reach-aware | `training_logs/videos/2026-06-11_15-48-51_mt4_reach_aware_stage0_128env_300iter_video/` |
| Stage 0 post-latch entry | `training_logs/videos/2026-06-11_15-55-46_mt4_reach_aware_stage0_post_latch_entry_128env_300iter_video/` |
| Stage 0 standoff2cm | `training_logs/videos/2026-06-11_16-51-01_mt4_reach_aware_stage0_standoff2cm_128env_600iter_video/` |
| Stage 0 entrygate | `training_logs/videos/2026-06-11_17-14-59_mt4_reach_aware_stage0_entrygate_128env_600iter/` |
| Stage 1 9-cell plane | `training_logs/videos/2026-06-11_20-14-22_mt4_coordinate_plane_9cell_xy035_center035_128env_600iter/` |

## 대표 그래프와 CSV

| 단계 | 위치 | 내용 |
| --- | --- | --- |
| Stage 0 workspace-entry | `training_logs/figures/20260610_222412_stage0_error.png` | error curve |
| Stage 0 workspace-entry | `training_logs/figures/20260610_222412_stage0_reward.png` | reward curve |
| Stage 0 workspace-entry | `training_logs/figures/20260610_222412_stage0_success_workspace.png` | success/workspace curve |
| Stage 0 workspace-entry | `training_logs/figures/20260610_222412_stage0_visibility.png` | visibility curve |
| Stage 1 seq9 | `logs/plots/20260612_seq9_5success/metric_summary.csv` | metric summary |
| Stage 1 seq9 | `logs/plots/20260612_seq9_5success/region_mastery.csv` | region mastery snapshot |
| Stage 1 seq9 | `logs/plots/20260612_seq9_5success/success_mastery_curve.png` | success/mastery curve |
| Stage 1 seq9 | `logs/plots/20260612_seq9_5success/region_progress_curve.png` | region progress |
| Stage 1 seq9 | `logs/plots/20260612_seq9_5success/region_success_counts.png` | region success counts |

## 최신 결과를 볼 때 확인할 파일

1. 최신 판단: `docs/CURRENT_BASELINE.md`
2. 누적 학습 표: `docs/TRAINING_HISTORY.md`
3. 상세 region/result 표: `docs/records/training/20260614_stage1_region16_17_radius_ladder_analysis.md`
4. camera audit 상세: `docs/records/training/20260614_stage1_cameraaudit_operating_workspace_analysis.md`
5. 산출물 위치: 현재 파일
