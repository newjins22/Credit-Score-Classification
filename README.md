# Credit Score Classification with TabNet


### 프로젝트 개요

본 프로젝트는 고객의 금융 활동 데이터를 바탕으로 신용 등급(Good, Standard, Poor)을 분류하는 딥러닝 모델을 구축하는 것을 목표로 합니다. 단순한 성능 수치보다는 데이터 전처리의 인과 관계, EDA를 통한 변수 해석, 모델 선택의 타당성에 초점을 맞추어 진행되었습니다.



### 주요 목표

1. 데이터 이해: EDA를 통해 신용 등급을 결정짓는 핵심 지표 발굴
2. 성능 개선: 정형 데이터 특화 모델인 TabNet 도입 및 하이퍼파라미터 최적화
3. 설명력 강화: 파생 변수 생성 및 인코딩 전략의 논리적 근거 제시



### EDA (Exploratory Data Analysis)

데이터의 특성을 파악하기 위해 시각화를 진행하였으며, 주요 인사이트는 다음과 같습니다.

1. 타겟 불균형 확인: 전체 데이터 중 Standard 등급이 53.2%로 가장 높으며 클래스 간 비중 차이가 존재함
2. 부채의 변별력: 미결제 부채(Outstanding_Debt)가 높을수록 신용 등급이 낮아지는 뚜렷한 경향성 확인
3. 연체의 영향: 연체 횟수(Num_of_Delayed_Payment)는 등급의 상한선을 결정하는 결정적 요인임
4. 비선형성 포착: 변수 간 상관관계가 낮음에도 불구하고 특정 조합에서 등급이 나뉘는 비선형적 패턴 확인

<img width="1389" height="985" alt="image" src="https://github.com/user-attachments/assets/8c466c70-15b2-4532-ae6f-00f946aeafe1" />




### 데이터 전처리 및 피처 엔지니어링

1. 식별자 제거: 예측에 무의미한 ID, Name, SSN 등의 컬럼 제거
2. 도메인 기반 파생 변수 생성

* debt_to_income: 부채 대비 소득 비율을 계산하여 실질적인 상환 능력 지표화
* delayed_interest: 연체 횟수와 이자율을 곱해 연체 리스크 가중치 산출
* loan_count: 쉼표로 구분된 대출 종류 텍스트를 수치화하여 대출 규모 정보 보존

3. 인코딩 최적화: 범주형 변수를 One-Hot Encoding으로 처리하여 TabNet이 변수 간의 수치적 서열 오해 없이 비선형 관계를 학습하도록 유도



### 모델링: TabNet (Deep Learning)

정형 데이터 학습에 최적화된 TabNet 모델을 선택했습니다.

1. 모델 선택 이유

* Attention 메커니즘: 각 단계에서 어떤 피처에 집중할지 스스로 학습하여 의사결정 나무의 장점과 딥러닝의 유연성을 결합
* Feature Interaction: 수동으로 찾기 힘든 피처 간의 복잡한 상호작용 학습 가능

2. 하이퍼파라미터 학습 전략

* Optimizer: AdamW (Weight Decay 적용으로 과적합 방지)
* Scheduler: CosineAnnealingLR을 통한 학습률 최적화
* Class Weights: 타겟 불균형 해소를 위해 손실 함수에 가중치 부여
* Regularization: Patience=15 설정을 통한 Early Stopping 적용



### 성능 결과 (Validation Score)

* Validation Accuracy: 0.7776 (77.8%)

[Classification Report]

```text
              precision    recall  f1-score   support

        Good       0.66      0.89      0.76      3566
        Poor       0.73      0.90      0.80      5799
    Standard       0.89      0.67      0.77     10635

    accuracy                           0.78     20000

```

* 해석: 클래스 불균형에도 불구하고 weights 조정을 통해 상대적으로 비중이 적은 Good과 Poor 클래스의 재현율(Recall)을 각각 89%, 90%까지 확보했습니다.



### 개선사항 및 결론

1. 모델 교체 효과: 기존 머신러닝 모델 대비 TabNet을 사용했을 때 변수 간 비선형 조합을 더 효과적으로 포착함을 확인했습니다.
2. 인코딩의 중요성: Label Encoding보다 One-Hot Encoding이 TabNet의 구조적 특성에 더 적합하며 성능 향상에 기여함을 실증했습니다.
3. 도메인 지식의 활용: 단순 데이터 입력보다 부채 대비 소득 비율 등 금융 도메인 지식을 반영한 피처 생성이 모델의 변별력을 크게 높였습니다.
