# kpop-mv-hit-prediction

K-pop 뮤직비디오의 오디오·비주얼·가사·아티스트 메타데이터로 유튜브 조회수 100만 이상("성공, Hit") 여부를 예측하는 이진 분류 프로젝트입니다. 전처리, 탐색적 분석, 변수 선정, 최종 분류 모델(CatBoost + SHAP)까지의 분석 과정을 다룹니다. (데이터 수집 단계는 이 저장소에 포함하지 않았습니다.)

## 파이프라인 개요

```
data_preparation  (수집 & 피처 추출 · 저장소에 포함하지 않음)
            ↓
data_preprocessing  (병합 & 중복 제거)
    merge_features
            ↓
    deduplication
            ↓
eda  (탐색적 분석)
    basic_eda
            ↓
    edge_case_analysis
            ↓
modeling  (변수 선정 & 최종 모델)
    variable_selection_decision_tree
            ↓
    feature_engineering_stepwise_selection
            ↓
    variable_refinement_model_selection
            ↓
    final_model
```

### data_preparation — 데이터 수집 & 피처 추출 (저장소 미포함)
음악 순위·유튜브 메타데이터·장르·가사 수집과, 오디오(valence/arousal 등)·영상(밝기·모션·색상) 피처 추출 단계입니다. 이 단계의 코드와 수집 데이터는 수집 대상 서비스의 이용약관과 저작권을 고려해 이 저장소에 포함하지 않았습니다.

### data_preprocessing — 병합 & 중복 제거
| 노트북 | 설명 |
|---|---|
| `merge_features.ipynb` | 곡 목록에 가사·장르 수집 결과와 특징 추출 결과 3개를 `video_id` 기준으로 붙여 하나의 통합본으로 병합 |
| `deduplication.ipynb` | 통합본에서 같은 곡의 중복 영상(곡명+아티스트 / `video_id` / 유통사 채널 기준)을 정리해 분석용 데이터 `data(drop_duplicated).csv` 생성 |

### eda — 탐색적 분석
| 노트북 | 설명 |
|---|---|
| `basic_eda.ipynb` | 가사·음향·영상 특징과 조회수의 관계, 장르별 차이, 장르 문법에서 벗어난 특이한 곡 탐색 |
| `edge_case_analysis.ipynb` | 특이 케이스(Case A~E) 분석. 발라드·댄스 장르별 가사 양·영어 비중과 히트율의 관계를 결론으로 정리 |

### modeling — 변수 선정 & 최종 모델
| 노트북 | 설명 |
|---|---|
| `variable_selection_decision_tree.ipynb` | 의사결정나무·RandomForest 기반 장르별 변수 중요도 탐색, 통합 RandomForest(원핫 인코딩)로 후보 변수 확인 |
| `feature_engineering_stepwise_selection.ipynb` | 파생변수 생성·원핫 인코딩 → Stepwise(전진+후진) 변수 선택 → 선정 변수로 첫 CatBoost 모델과 오답 패턴 확인 |
| `variable_refinement_model_selection.ipynb` | 발라드 변수 보완, 여러 알고리즘 비교(CatBoost 선택), FP 개선용 파생변수 추가 등 수동 변수 보완 과정 |
| `final_model.ipynb` | 최종 변수 22개로 CatBoost 학습 + SHAP 분석 (단독 실행 가능) |

## 최종 모델 결과

- **성능** (5-Fold 교차검증 기준): Accuracy 0.768 / Precision 0.811 / Recall 0.813 / F1 0.812 / AUC 0.820
- **SHAP 분석**: 상위 11개 변수 중 1위는 장르(`genre_dance`)이고, 이어서 영어 가사 특성(`speechiness_eng`, 2위), 솔로 발라드 지수(`solo_ballad`, 3위), 시청각 결합 특성(`avg_motion`, `visual_audio_energy`, `color_rgb`, `visual_power`, 4~7위)이 나타났습니다. 글로벌 가사 관련 `rap_globalization_tempo`, `korean_ratio`도 상위 11개에 포함됩니다.

## 사용 방법

```bash
pip install -r requirements.txt
```

각 노트북은 자신이 있는 폴더(`data_preprocessing/`, `eda/`, `modeling/`)에서 실행하는 것을 기준으로 데이터를 `../data/...` 상대 경로로 읽고 씁니다.

```
data/
├── raw/                                                 수집·피처 추출 결과 (저장소 미포함)
├── kpop_radar_2023_2025_dedup_merged_features.csv       병합 통합본 (deduplication 노트북의 입력)
└── data(drop_duplicated).csv                            분석용 데이터 (eda, modeling 노트북의 입력)
```

수집 데이터와 중간 산출물 CSV는 용량·개인정보·저작권을 고려해 이 저장소에 포함하지 않았습니다.
