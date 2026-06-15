# ⚽ AI 기반 축구 선수 스카우팅 분석 시스템

### 전술 요구에 맞는 선수를 정량 평가하는 AI 스카우팅 파이프라인

> **"영상에서 뽑은 공간 데이터 + 이벤트 데이터를 결합해, 전술 요구에 부합하는 선수를 정량 평가한다"**

---

## 🎯 프로젝트 개요

단순 통계 나열이 아닌, **전술 요구 → 측정 가능한 지표 → 후보 평가**로 이어지는
실무형 스카우팅 의사결정 파이프라인을 구현했습니다.

구단과 포지션, 전술 가중치만 바꾸면 **어떤 팀, 어떤 포지션의 영입 후보든 평가 가능**한 범용 시스템입니다.

| 항목 | 내용 |
|---|---|
| 적용 가능 구단 | 모든 구단 (가중치·브리프 교체만으로 즉시 적용) |
| 적용 가능 포지션 | 모든 필드 포지션 |
| 데이터 소스 | 이벤트 데이터(FBref/WhoScored) + 영상 위치 데이터(CV) |
| 핵심 차별점 | **영상 위치 데이터(Computer Vision) + 이벤트 데이터 결합** |

---

## 🏟️ 적용 예시 — Manchester United CM 앵커 영입

본 프로젝트의 데모 케이스로 **맨체스터 유나이티드의 중앙 미드필더 영입 시나리오**를 사용합니다.

| 항목 | 내용 |
|---|---|
| 대상 구단 | Manchester United |
| 감독 | Michael Carrick (2026년 5월 정식 선임) |
| 영입 포지션 | 중앙 미드필더 — 카세미루 이적 후 앵커 공백 |
| 분석 후보 | Mateus Fernandes, Sandro Tonali, Alex Scott, Adam Wharton, Mason Mount(대조군) |

---

## 🏗️ 시스템 구조

```
영상 입력
    ↓
[YOLO 선수 검출] → [ByteTrack 추적] → [팀 분류] → [호모그래피 좌표 변환]
    ↓
피치 좌표 CSV (frame, tracker_id, role, team, x, y)
    ↓
[위치 지표 계산]          [이벤트 데이터 (FBref)]
평균위치·커버범위·존점유율    인터셉트·태클·파울·출전시간
    ↓                          ↓
         [0~1 정규화 + 가중합]
                ↓
         후보별 적합도 점수 & 순위
```

---

## 📐 스카우팅 브리프 (설계 철학)

전술 분석을 바탕으로 **포지션 요구사항 → 측정 지표 → 가중치**를 도출합니다.
가중치는 브리프 교체만으로 다른 구단/포지션에 즉시 적용 가능합니다.

### 평가 자질 및 가중치 (맨유 CM 앵커 예시)

| 순위 | 자질 | 가중치 | 측정 방식 |
|---|---|---|---|
| 1 | 수비 안정성 | **30%** | 인터셉트·태클(이벤트) + 수비 위치(CV) |
| 2 | 압박 저항·템포 | **25%** | 파울(역산)·패스 성공률(이벤트) |
| 3 | 피지컬·활동량 | **20%** | 총 이동거리·스프린트(CV) + 출전시간(이벤트) |
| 4 | 볼 운반 | **15%** | Progressive carries(이벤트) |
| 5 | 볼 전진 | **10%** | Progressive passes(이벤트) |

> 상세 설계 문서 → [`brief/scouting_brief_manutd_cm.md`](brief/scouting_brief_manutd_cm.md)

---

## 📊 분석 결과 (맨유 CM 앵커 데모)

### 두 모델 비교 — 같은 선수, 다른 브리프

> 핵심 차별점: **동일한 평가 엔진에 가중치만 바꾸면 전혀 다른 후보가 도출된다**

| 선수 | 타깃 A 앵커 | 타깃 B 마이누 백업 | 해석 |
|---|---|---|---|
| Mateus Fernandes | 🥇 0.744 | 🥇 0.633 | 두 모델 모두 상위 — 올라운더 |
| Alex Scott | 2위 0.561 | 4위 0.492 | 체력·운반 강점 → 앵커에 유리 |
| Sandro Tonali | 3위 0.542 | 3위 0.536 | 두 모델 모두 중위 — 균형형 |
| Adam Wharton | 4위 0.539 | 2위 0.613 | 전진 만점 → 마이누 백업에 유리 |
| Mason Mount | 5위 0.198 | 5위 0.159 | 두 모델 모두 최하위 — 음성 대조군 검증 |

**와튼↔스콧의 순위 역전**이 설계 의도를 증명합니다.
앵커(수비30%)에선 체력·운반이 강한 스콧이 앞서지만,
마이누 백업(전진30%)에선 전진 패스 만점의 와튼이 역전합니다.

---

## 🛠️ 기술 스택

| 영역 | 기술 |
|---|---|
| 선수 검출 | YOLOv8 (Roboflow sports) |
| 다중 객체 추적 | ByteTrack (supervision) |
| 팀 분류 | SigLIP + UMAP + KMeans |
| 피치 캘리브레이션 | 호모그래피 (ViewTransformer) |
| 이벤트 데이터 | soccerdata + FBref |
| 분석·시각화 | pandas, numpy, matplotlib |
| 개발 환경 | Python 3.11, Apple Silicon MPS |

---

## 📁 파일 구조

```
footballer-scouting/
├── README.md
├── brief/
│   └── scouting_brief_manutd_cm.md   # 스카우팅 설계 문서 (맨유 CM 예시)
├── notebooks/
│   ├── 01_position_analysis.ipynb    # 위치 데이터 분석 (히트맵·산점도)
│   └── 02_scoring_model.ipynb        # 적합도 점수 모델 (레이더·바 차트)
├── scripts/
│   └── extract_coordinates.py        # CV 좌표 추출 스크립트
└── requirements.txt
```

---

## 🚀 실행 방법

```bash
# 1. 환경 설정
conda activate dl_study
pip install -r requirements.txt

# 2. 좌표 추출 (영상 → CSV)
python scripts/extract_coordinates.py \
  --source_video_path <영상경로> \
  --output_csv_path data/coordinates.csv \
  --device mps

# 3. 노트북 실행
jupyter lab
# notebooks/01_position_analysis.ipynb → 02_scoring_model.ipynb 순서로 실행
```

---

## ⚠️ 한계 및 향후 계획

**현재 한계**
1. CV 데이터는 분데스리가 샘플 기준 — 후보 선수 영상 확보 후 재측정 필요
2. 5명 기준 MinMax 정규화 — 리그 전체 분포 기준으로 보강 시 더 견고해짐

**향후 계획**
- [ ] API 자동화 — 데이터 수집 파이프라인 고도화
- [ ] LangChain 연동 — 선수 관련 외부 정보(뉴스·이적 루머) 자동 검색
- [ ] LangGraph Agent — 브리프 입력부터 리포트 생성까지 전 과정 자동화
- [ ] 후보 선수 영상 확보 → CV 지표와 이벤트 지표 결합
- [ ] 다양한 리그·포지션으로 파이프라인 일반화

---

## 👤 About

비CS 전공(경영·마케팅·호텔경영) → AI/ML 전환 중.
축구부 출신(고등학교)의 스포츠 도메인 지식과 AI 기술을 결합해 **데이터와 현장 눈을 동시에 읽는 스카우트**를 목표로 합니다.

> 관련 문의: GitHub Issues 또는 PR 환영합니다.