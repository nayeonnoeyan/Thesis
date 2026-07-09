# XAI 기반 연체 경험 차주의 신용회복 및 장기 부실화 결정요인 분석
**An Analysis of Credit Recovery and Long-term Default Determinants among Delinquent Borrowers Using Explainable Artificial Intelligence**

> LightGBM + SHAP 기반 금융 데이터 분석 | 졸업논문 구현 코드

---

## 📌 프로젝트 개요

기존 신용위험 연구는 연체 발생 여부 예측에 집중되어 왔으나, 본 연구는 **이미 연체를 경험한 차주**를 대상으로 신용을 회복하는 집단과 장기 부실 상태로 고착되는 집단을 구분하고 그 결정요인을 분석합니다. XAI 기법인 SHAP을 활용하여 모델의 예측 결과를 해석하고, 금융기관의 조기경보 시스템 개선에 활용 가능한 시사점을 도출하였습니다.

---

## 🗂️ 데이터

- **출처**: AI Hub 금융 합성 데이터 (2022년 12월 기준 개인 CB 데이터)
- **분석 대상**: 최장 연체일수 1일 이상인 차주 151,938명
- **특수값 처리**: 8888888.8(미보유/결측) → NaN, -9(정보 없음) → 별도 카테고리

### 종속변수

| 구분 | 기준 | 비율 |
|------|------|------|
| 단기 회복군 (0) | 최장 연체일수 300일 미만 | 57.4% |
| 장기 고착군 (1) | 최장 연체일수 300일 이상 | 42.6% |

### 독립변수 (총 20개)

| 카테고리 | 변수 |
|----------|------|
| 사회경제적 배경 | Gender, Age_Band, House_Size, Home_Price, Asset_Value, Job_History_3Y, Addr_History_3Y |
| 신용 및 대출 레버리지 | Loan_Count, Loan_Amt, Loan_Inst_Cnt, Savings_Loan_Cnt, Bank_Loan_Cnt, Card_Total_Limit, Card_Limit_Usage, CA_Usage_Rate |
| 이용 행태 및 결제 이력 | Card_Lump_Amt, Card_Install_Amt, Card_Use_3M, Revolving_Cards, Overdue_Repay_Amt |

---

## 🔧 분석 파이프라인

```
데이터 로드 (Polars)
    ↓
결측치 및 특수값 처리
    ↓
종속변수 생성 (Max_Overdue_Days 기준)
    ↓
IQR 클리핑 (로지스틱 회귀용)
    ↓
다중공선성 검정 (VIF)
    ↓
Train/Test 분할 (Stratified Sampling, 8:2)
    ↓
모델 학습 (LightGBM / Logistic Regression)
    ↓
성능 비교 (AUC, F1-score, Confusion Matrix, ROC Curve)
    ↓
SHAP 분석 (Summary Plot, Waterfall Plot)
    ↓
t-검정 기반 통계적 유의성 검증
```

---

## 🤖 모델

### LightGBM (최종 모델)

| 하이퍼파라미터 | 값 |
|---|---|
| learning_rate | 0.03 |
| max_depth | 8 |
| num_leaves | 127 |
| subsample | 0.7 |
| colsample_bytree | 0.7 |
| reg_alpha | 0.1 |
| reg_lambda | 0.1 |

### Logistic Regression (비교 모델)

| 하이퍼파라미터 | 값 |
|---|---|
| C (규제 강도) | 0.01 |

---

## 📈 모델 성능 비교

| 모델 | AUC | F1-score |
|------|-----|----------|
| **LightGBM** | **88.62%** | **77.18%** |
| Logistic Regression | 85.22% | 74.07% |

---

## 🔍 SHAP 분석 주요 결과

| 변수 | 방향 | 해석 |
|------|------|------|
| Card_Lump_Amt (신용카드 일시불 이용금액) | 높을수록 → 단기 회복 | 즉시 결제 능력이 있는 차주는 회복 가능성 높음 |
| Card_Limit_Usage (신용카드 한도 소진율) | 높을수록 → 장기 고착 | 유동성 한계에 다다른 차주는 부실 위험 큼 |
| Overdue_Repay_Amt (총 연체 상환금액) | 높을수록 → 장기 고착 | 누적된 연체 부담이 클수록 회복 어려움 |
| Age_Band (연령대) | 30~50대 → 장기 고착 | 생애주기상 지출 집중 시기로 금융 부담 큼 |
| Addr_History_3Y (자택 주소 이력 건수) | 많을수록 → 장기 고착 | 잦은 이사 = 주거 불안정 신호 |
| Job_History_3Y (직장 이력 건수) | 많을수록 → 단기 회복 | 지속적인 경제활동 참여 반영 |

---

## 🛠️ 사용 라이브러리

```python
polars / pandas        # 데이터 로드 및 처리
numpy                  # 수치 연산
scikit-learn           # 모델 학습, 평가, 데이터 분할
lightgbm               # LightGBM 분류 모델
shap                   # XAI 분석
statsmodels            # VIF 다중공선성 검정
matplotlib             # 시각화
scipy                  # 통계 검정
```

---

## 📂 파일 구성

```
Thesis/
├── credit_default_xai.ipynb    # 전체 분석 코드
└── README.md
```

> 데이터는 AI Hub에서 별도 신청 후 사용 가능합니다.
> [AI Hub 금융 합성 데이터](https://aihub.or.kr/)
