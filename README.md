# DACON 제주 특산물  가격 예측 AI 경진대회 (연습)

### 참조: https://dacon.io/competitions/official/236176/codeshare/9381?page=1&dtype=recent

private 1등을 하신 저어어어어엉 님의 코드를 실험하고 개인적인 추가 실험 몇 가지를 진행해봤습니다.

task: 2019년 1월 1일~2023년 3월 3일까지의 품목별 일별 가격 데이터를 가지고 2023년 3월 4일-2023년 3월 31일까지의 각 품목의 가격을 예측해보는 문제입니다.

### data:

1. 2019년 1월 1일~2023년 3월 3일까지 일별 가격 및 유통량 데이터 - train data
   
   ![train_data.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/train_data.png)

2. 2019년 1월~2023년 2월까지 월별 국제 수출/수입 금액 및 중량 데이터 - 보조 covariate data

   ![international_trade.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/international_trade.png)

3. 2023년 3월 4일~2023년 3월 31일까지의 품목, 유통 법인, 지역 코드 데이터 - test data

   ![test_data.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/test_data.png)


### model:

저어어어어엉님이 구상하신대로 감귤 외의 품목은 CatBoostRegressor와 XGBoostRegressor의 앙상블 모델로 가격을 예측하였고 감귤 품목의 경우 CatBoostRegressor와 XGBoostRegressor의 앙상블 모델과 또 하나의 CatBoostRegressor 모델로 이루어진 앙상블 모델로 가격을 예측하였습니다. 시계열 데이터지만 테이블 데이터로 상정하여 분석하였습니다.

### 추가 실험:

저어어어어엉님이 공유하신 코드에는 cross validation 과정은 포함되지 않았고 이미 time series cross validation 과정을 거쳐 최적화된 파라미터로 두 앙상블 모델을 결정 지으신 상태였습니다. 저는 time series cross validation을 사용하지 않고, 아무래도 seasonality가 두드러지는 특산물 가격 특성상 2019, 2020, 2021, 2022년 각 해의 3월 가격 데이터를 validation set으로 정하여 4-fold cross validation으로 CatBoostRegressor와 XGBoostRegressor의 hyper parameter들을 튜닝하였습니다. 이 과정에서 optuna 라이브러리를 활용하여 편리하게 최적화할 수 있었습니다.

또 하나 추가적으로 제가 진행한 사항은 국제 무역 데이터의 활용입니다. 국제 무역의 경계가 없다시피 한 요즘 제주 특산물이라도 비슷한 품목의 가격이나 시세에 영향을 받지 않을 수 없다고 생각했습니다. 감귤, 양배추 및 당근은 해당하는 품목이 국제 무역 데이터에도 존재하여 각 월 시세를 학습 데이터에 매핑하였고 브로콜리는 "꽃양배추와 브로콜리(broccoli)', 무는 상응하는 품목이 없어 None으로 두었습니다.

### results:


아쉽게도 저어어어어엉님의 public score 654점보다 다소 낮은 public score 693을 기록했습니다. 국제 무역 데이터만을 보조 covariate 데이터로 사용했을 때 652점의 점수를 올리며 진전이 있었지만 점진적으로 validation set과 hyper parameter tuning 과정에서 성능이 내려가는 결과로 이어졌다고 생각됩니다. 혹은 모델이 과거년도의 3월에 너무 과적합 된 나머지 다소 다른 경향을 보인 2023년 3월의 가격 예측을 잘 하지 못했을수도 있다고 생각됩니다.

![submission.png](https://github.com/wschung1113/jeju_specialty/blob/main/images/submission.png)