### 1.	MySQL계정에 필요한 권한 확인
GRANT SELECT, ALTER ON schema_name.* TO 'user_name'@'%';
GRANT SELECT, EXECUTE ON sys.* TO 'user_name'@'%';


### 2.	Iris 학습 데이터 및 테스트 데이터 로딩
- [Iris 데이터 ](http://archive.ics.uci.edu/ml/datasets/Iris) 
HeatWave Machine Learning (ML)의 기능을 사용하여 홍채꽃(Iris)의 종을 분류하는 머신러닝 프로젝트를 실습합니다.
