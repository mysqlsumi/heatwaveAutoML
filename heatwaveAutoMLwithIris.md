### 1.	MySQL계정에 필요한 권한 확인
```
GRANT SELECT, ALTER ON schema_name.* TO 'user_name'@'%';
GRANT SELECT, EXECUTE ON sys.* TO 'user_name'@'%';
```


### 2.	Iris 학습 데이터 및 테스트 데이터 로딩
- HeatWave Machine Learning (ML)의 기능을 사용하여 붓꽃(Iris)의 종을 분류하는 머신러닝 프로젝트를 실습합니다.
![Alt text](image.png)

- 아래 링크를 클릭해서 해당 쿼리를 mysql shell 또는 클라언트를 이용해서 MDS에 적용


[Iris 데이터 ](https://oracle-livelabs.github.io/database/heatwave-machine-learning/do-heatwave-ml/files/iris-ml-data.txt) 

```
mysqlsh admin@<mds ip address>:3306

use ml_data;
show tables;
```
- 데이터가 정상적으로 로딩되었는지 확인합니다. ml_data 스키마에 다음 3개 테이블이 있습니다.


![Alt text](image-1.png)


### 3. ML_TRAIN 을 사용하여 모델을 학습합니다. 이는 분류 데이터 셋이므로 classification을 사용하여 분류 모델을 만듭니다.
- 학습 작업이 완료되면 모델 핸들이 @iris_model세션 변수에 할당되고 모델은 모델 카탈로그에 저장됩니다. 
```
CALL sys.ML_TRAIN('ml_data.iris_train', 'class', NULL, @iris_model);
```

![Alt text](image-2.png)


- 다음 쿼리를 사용하여 모델 카탈로그의 항목을 확인합니다. user1을 MDS의 관리 계정(ex. admin)으로 바꿉니다 .
```
select @iris_model;

SELECT model_id, model_handle, train_table_name FROM ML_SCHEMA_admin.MODEL_CATALOG;
```


![Alt text](image-3.png)



### 4. ML_MODEL_LOAD 루틴을 사용하여 모델을 HeatWave AutoML에 로드합니다.
```
CALL sys.ML_MODEL_LOAD(@iris_model, NULL);
```


![Alt text](image-4.png)


모델을 사용하려면 먼저 모델을 HeatWave로 로드해야 합니다. 언로드하거나 HeatWave 클러스터를 다시 시작하기 전까지 모델은 로딩된 상태로 유지됩니다.



### 5. ML_PREDICT_ROW 루틴을 사용하여 단일 row(행) 데이터에 대한 예측을 수행합니다. 

```
SELECT sys.ML_PREDICT_ROW(JSON_OBJECT("sepal length", 7.3, "sepal width", 2.9, "petal length", 6.3, "petal width", 1.8), @iris_model, NULL);
```

![predic_row](image-11.png)


제공된 특징 값을 바탕으로 모델은 이 iris 종류가 “Iris-virginica” 분류의 임을 예측합니다. 예측을 수행하는 데 사용되는 특징 값도 표시됩니다.



### 6. ML_EXPLAIN_ROW 루틴을 사용하여 데이터 행에 대한 설명을 생성하여 예측이 어떻게 이루어졌는지 이해할 수 있습니다.

```
SELECT sys.ML_EXPLAIN_ROW(JSON_OBJECT("sepal length", 7.3, "sepal width", 2.9, "petal length", 6.3, "petal width", 1.8), @iris_model,    JSON_OBJECT('prediction_explainer', 'permutation_importance'));
```


![explain_row](image-12.png)



### 7. ML_PREDICT_TABLE 루틴을 사용하여 테이블 데이터에 대한 예측을 수행합니다.
```
CALL sys.ML_PREDICT_TABLE('ml_data.iris_test', @iris_model,'ml_data.iris_predictions', NULL);
SELECT * FROM ml_data.iris_predictions LIMIT 3;
```


![predict table](image-9.png)



### 8. 테이블 예측에 대한 해석을 수행합니다.
```
CALL sys.ML_EXPLAIN_TABLE('ml_data.iris_test',@iris_model,'ml_data.iris_explanations', JSON_OBJECT('prediction_explainer', 'permutation_importance'));
SELECT * FROM ml_data.iris_explanations;
```


![explain table](image-8.png)



### 9. 머신 러닝 모델의 정확도를 평가할 수 있는 score를 계산하고 모델을 언로딩합니다.
```
CALL sys.ML_SCORE('ml_data.iris_validate', 'class', @iris_model, 'balanced_accuracy', @score, NULL);

SELECT @score;

CALL sys.ML_MODEL_UNLOAD(@iris_model);
```

![score](image-7.png)


![unload model](image-10.png)



## 참고자료
- [HeatWave Quick Start ](https://dev.mysql.com/doc/heatwave/en/mys-hwaml-iris-example.html)