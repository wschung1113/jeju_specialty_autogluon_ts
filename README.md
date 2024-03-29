# DACON 제주 특산물  가격 예측 AI 경진대회 (연습) with AutoGluon-ts library

### reference

1. https://dacon.io/competitions/official/236176/codeshare/9188?page=3&dtype=recent

2. https://auto.gluon.ai/stable/tutorials/timeseries/forecasting-indepth.html

basslibrary 님의 베이스 코드를 기반으로 개인적인 추가 실험 몇 가지를 진행해봤습니다.

### task

2019년 1월 1일~2023년 3월 3일까지의 품목별 일별 가격 데이터를 가지고 2023년 3월 4일-2023년 3월 31일까지의 각 품목의 가격을 예측해보는 문제입니다.

### data:

1. 2019년 1월 1일~2023년 3월 3일까지 일별 가격 및 유통량 데이터 - train data
   
   ![train_data.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/train_data.png)

2. 2019년 1월~2023년 2월까지 월별 국제 수출/수입 금액 및 중량 데이터 - 보조 covariate data

   ![international_trade.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/international_trade.png)

3. 2023년 3월 4일~2023년 3월 31일까지의 품목, 유통 법인, 지역 코드 데이터 - test data

   ![test_data.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/test_data.png)


### model:

Preset별로 다양한 모델을 손쉽게 접할 수 있는 AutoML library입니다. Baseline preset의 경우 통계 모델과 tree 기반 모델 등 베이스 라인 모델 위주로 구성되어 있습니다. Medium과 high quality preset으로 넘어갈 수록 deep learning 모델들이 추가되며 (Temporal Fusion Transformer, PatchTST, DLinear) best quality preset의 경우 high quality preset과 모델 구성은 같지만 더 많은 cross-validation window를 가집니다. 현재 시계열 예측의 SOTA 모델부터 top-down 형식으로 공부하고 있어 PatchTST와 DLinear 모델에 관심을 가지고 하이퍼파라미터 커스터마이제이션 (look-back window 길이, transformer 인코더 레이어 수 등) 또한 진행하였습니다.

### 추가 실험:

1. 첫번째 실험으로는 각 시계열을 item_id (품목, 유통 법인, 지역으로 분류; i.e., TG_A_S (감귤, A 유통 법인, 서귀포 시))로 나눈 후 target 예측 값을 품목의 가격으로 두고 past covariate (예측 시점 이전까지만 알 수 있는 covariate)으로 품목별 유통량을 사용하였습니다. Model preset은 baseline preset으로 사용하였고 모델 학습 후 .refit_full() 메소드를 통해 검증 데이터셋에 대하여 다시 한번 모델 피팅을 진행하였습니다.

2. 두번째 실험으로는 위 참조 '2.' autogluon-ts 공식 문서를 읽으며 가능한 부가 옵션을 모두 활용하였습니다. 

   a. 첫번째는 static features를 활용하는 것이었습니다. Static features란 시계열의 변하지 않는 속성입니다. 예를 들어 어느 감귤 가격 시계열의 유통 법인과 지역은 변하지 않습니다. 첫번째 실험에서는 품목, 유통 법인, 지역별로 시계열을 모두 나누어 학습하였는데 이 경우 동일 품목 타 유통 법인이나 같은 지역 내에서 발생한 가격에 미치는 영향을 반영할 수 없다고 생각해서 static feature들을 활용해보았습니다.

   b. known & past covariate 들을 활용하였습니다. Known covariate의 경우 예측 시점에도 알 수 있는 covariate들을 말하며 (날짜, 휴일 여부 등) past covariate은 예측 시점 이전까지만 알 수 있는 데이터입니다 (타 품목 당일 가격, 유통량 등)

   c. Custom 검증 데이터셋으로 모델을 학습하였습니다. 2023년 3월의 가격을 예측하는만큼 2019, 2020, 2021, 2022년 3월을 검증 데이터셋으로 구성하였습니다.

   d. 주어진 model preset을 사용하지 않고 long term timeseries forecasting SOTA 모델에 속하는 DLinear과 PatchTST 모델을 사용했습니다. 베이스라인으로 DeepAR과 Theta 모델 또한 학습하였습니다. DLinear 모델의 경우 default hyperparameter로 학습하였고 PatchTST 모델의 경우 transformer 인코더 레이어를 6으로 확장하고 look-back window를 특산품 가격의 일년 계절성을 생각하여 365로 학습했습니다. Patch length나 stride, 임베딩 크기는 default 세팅을 사용하였습니다.

### results:

Public score는 730점으로 하나의 past covariate만을 사용한 첫번째 실험 방식이 제일 나은 성능을 보였습니다. Tree 기반 regression 모델을 사용해서 시계열을 테이블 데이터처럼 분석한 저의 다른 프로젝트는 650점대의 public score가 나온 것에 비해 저조하다고 할 수 있습니다.

![submission_autogluon_ts.png](https://github.com/wschung1113/jeju_specialty_autogluon_ts/blob/main/submission_autogluon_ts.png)

일단 두번째 풀옵션 실험에 비해 첫번째 실험의 결과가 월등히 좋았던 것에 대한 하나의 이유를 말하자면 너무 많은 feature를 제공한 것이라고 생각합니다. 시계열의 각 시점 가격과 covariate들을 테이블의 한 행으로 상정하여 분석하는 CatBoostRegressor나 XGBoostRegressor의 경우 제공되는 feature는 이번 프로젝트에서 많아야 30~40개였습니다. 그러나 시계열의 경우, 각 look-back window의 시점 하나하나에 각 시점의 feature 차원을 곱한 값이 총 feature volume과 상응한다고 생각하여 섣불리 각 시점의 covariate 차원을 늘리거나 look-back window를 늘리면 안되겠다는 생각을 했습니다. 또한 모델 복잡도를 높여 많은 feature들을 소화할 수 있는 크기로 만들어야 겠다고 생각했습니다.

다음에 시계열 분석 과제가 주어진다면 시계열 모델과 regression 모델 모두 사용하는 앙상블 모델 쪽으로 실험할 것 같습니다. 앙상블의 시계열 모델은 autoregression에 집중하도록 covariate을 많이 두지 않고 예측을 하게끔 하고 regressor 모델은 각 시점의 feature들 간의 관계성에 집중할 듯 싶습니다.

마지막으로 custom validation set을 과거년도의 3월들에만 포커싱했던 것이 모델이 되려 과거 3월들에 과적합되어 안좋은 성능을 나타내지 않았을까 생각해봅니다.

### .refit_full() method에 대한 문의글 작성

https://github.com/autogluon/autogluon/discussions/4016
