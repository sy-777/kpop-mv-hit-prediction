# kpop-mv-hit-prediction

K-pop 뮤직비디오의 오디오·비주얼·가사·아티스트 메타데이터로 유튜브 조회수 "성공(Hit)" 여부를 예측하는 프로젝트입니다. 크롤링부터 피처 추출, 전처리, 탐색적 분석, 변수 선정, 최종 분류 모델(CatBoost + SHAP)까지 전체 파이프라인을 다룹니다.

## 파이프라인 개요

```
데이터 수집 & 피처 추출               분석 파이프라인 (00 ~ 05)
─────────────────────────           ──────────────────────────────────────
crawling_kpop_rankings_...     ┐
crawling_genre_lyrics                │
feature_extraction_audio_*      ├──▶  00_preprocessing
feature_extraction_visual      ┘        ↓
                                       01_basic_eda
                                         ↓
                                       02_edge_case_analysis
                                         ↓
                                       03_gini_lorenz_curve
                                         ↓
                                       04_variable_selection_decision_tree
                                         ↓
                                       05_variable_selection_final_model
```

### 데이터 수집 & 피처 추출
| 노트북 | 설명 |
|---|---|
| `crawling_kpop_rankings_youtube_metadata.ipynb` | 음악 순위 사이트에서 2023~2025년 월간 유튜브 순위 수집 + YouTube Data API로 조회수·좋아요·게시일 등 메타데이터 보강 |
| `crawling_genre_lyrics.ipynb` | 곡명+아티스트로 음원 사이트에서 장르·가사 크롤링 |
| `feature_extraction_audio_valence_arousal.ipynb` | Essentia로 valence/arousal 등 오디오 감성 피처 추출 |
| `feature_extraction_audio_additional.ipynb` | 추가 청각 피처 추출 |
| `feature_extraction_visual.ipynb` | 영상에서 밝기·모션·색상 등 시각 피처 추출 |

### 분석 파이프라인
| 노트북 | 설명 |
|---|---|
| `00_preprocessing.ipynb` | 2024 원본 정리 → 중복 처리(songName+artists / video_id / 유통사 채널 기준) → 2023,2025 데이터 추가 → 최종 병합 |
| `01_basic_eda.ipynb` | 기본 탐색적 데이터 분석 |
| `02_edge_case_analysis.ipynb` | 특이 케이스 분석 |
| `03_gini_lorenz_curve.ipynb` | 장르별 조회수 편중도(지니계수·로렌츠 곡선) 분석 |
| `04_variable_selection_decision_tree.ipynb` | 의사결정나무·RandomForest 기반 장르별 변수 중요도 탐색 |
| `05_variable_selection_final_model.ipynb` | Stepwise 변수 선택 → 수동 파생변수 보완 → 최종 CatBoost 모델 + SHAP 분석 |

`archive/`에는 최종 노트북으로 정리되기 전의 원본 탐색 기록(초안, 모델링 시행착오)이 보관돼 있습니다.

## 최종 모델 결과

- **성능**: Accuracy 0.768 / Precision 0.811 / Recall 0.813 / F1 0.812 / AUC 0.820
- **SHAP 분석**: 상위 중요 변수는 장르(`genre_dance`), 영어/글로벌 지향 가사 특성(`speechiness_eng`, `rap_globalization_tempo`, `korean_ratio`), 시청각 결합 특성(`avg_motion`, `visual_audio_energy`, `visual_power`, `color_rgb`) 순으로 나타났습니다.

## 사용 방법

```bash
pip install -r requirements.txt
```

각 노트북은 원래 Google Colab(`/content/...` 경로) 환경에서 작성되었습니다. 로컬에서 실행하려면 노트북 상단의 `pd.read_csv(...)` 경로를 실제 데이터 파일 위치로 바꿔주세요. 크롤링에 필요한 YouTube Data API 키는 환경변수 `YOUTUBE_API_KEY`로 설정합니다.

크롤링 원본 데이터, 중간 산출물 CSV는 용량·개인정보 문제로 이 저장소에 포함하지 않았습니다.
