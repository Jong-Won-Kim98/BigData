# BigData
----
<강원테크 SW융합교육>

데이터 엔지니어링?
: 빅데이터 기반 의사결정을 만들기 위한 인프라 구성, 인사이트 추출

GIGO(Garbage In Garbage Out)
: 좋은 데이터를 수집하고 잘 관리하고 처리하는 것이 훨씬 효율적이다

과거 데이터 아키텍쳐(문제점)
1. 구축 시스템이 비싸다
2. 데이터의 용도가 정해져있다
3. 데이터 수집처가 일정하다

* ETL
: 데이터의 형식이 지정, 변동이 없는 환경에서의 데이터 파이프 라인
- Extract(추출): 기존의 DB에서 데이터를 가져온다
- Transform(변환): 미리 정해 놓은 스키마에 맞게 데이터를 변환
- Load(적재): 변환이 완료된 데이터를 원하는 스키마에 INSERT하는 과정

현재 데이터 아키텍쳐
1. 다양한 데이터의 형식(스키마 정의가 어렵다)
2. 저렴해진 컴퓨터

* ELT
: 데이터를 추출한 후 선 저장하고 쓰임새에 따라 변환

데이터 아키텍처 분야
- 소스: 비즈니스와 운영 데이터 생성
- 수집 및 변환: ELT
- 저장: 데이터를 처리 시스템이 쓸 수 있도록 저장, 비용과 확장성 면으로 최적화
- 과거, 예측: 저장된 과게 데이터를 통해 인사이트 생성(Query), 쿼리를 실행하고 필요시 분산 처리(Processing), 과거에 일어난 일, 미래에 일어날 일(Machine Learning)
- 출력: 데이터 분석을 내,외부에 제공, 데이터 모델을 운영 시스템에 적용

데이터의 흐름
<img src = "Dataflow.png">

* Batch Processing(한꺼번)
Batch: 일괄, Processing: 처리
- 많은 양의 데이터를 정해진 시간에 한꺼번에 처리
- 전통적으로 사용한 데이터 처리 방법
- 실시간성을 보장하지 않아도 될 때
- 무거운 처리를 할 때
- 마이크로 배치: 데이터를 조금씩 모아서 프로세싱하는 방식(Spark Streaming)

* Stream Processing(실시간)
- 실시간으로 쏟아지는 데이터를 계속 처리하는 것
- 이벤트가 생길 때, 데이터가 들어올 때 마다 처리
- 불규칙적으로 데이터가 들어오는 환경

---
실습 환경 구비

설치 프로그램
1. Python(anaconda)
2. java 설치(oracle jdk 11) - Spark 구성 언어
3. Hadoop winutils 2.7.7 - Hadoop 환경 간접 설정
4. apache spark 

환경 변수 설정
1. PYSPARK_PYTHON
2. JAVA_HOME
3. HADOOP_HOME
4. SPARK_HOME
---

#### Spark를 이용한 학생 수 카운트

* SparkConf: Spark 설정 옵션 객체, 주로 SparkContext 설정
   - setMaster: Spark가 실행될 위치 설청, local 또는 분산(HDFS) 등을 사용
   - setAppName: 스파크에서 작업할 어플리케이션의 이름, 웹 환경(Spark UI)에서 확인이 가능하다
* SparkContext: Spark 클러스터와 연결 시켜주는 객체
   - Spark의 모든 기능에 접근할 수 있는 시작점
   - Spark는 분산 환경에서 동작하기 때문에 Driver Program을 구동시키기 위해서는 SparkContext가 필요하다
   - SparkContext는 프로그램당 하나만 만들수 있고, 사용후에는 종료해야 한다

* SparkContext 작동 과정
   - SparkContext 객체의 내부는 자바로 동작하는 Py4j의 SparkContext와 소켓을 통해 연결된다.
   - Py4j란 Python되어 있는 코드를 Spark에서 구동 가능한 java 형태의 스칼라로 변환
   - RDD를 만들 수 있다.(Spark에서 사용하는 데이터 구조)

```Python
from pyspark import SparkConf, SparkContext

conf = SparkConf().setMaster("local").setAppName("country-student-counts")

sc = SparkContext(conf=conf)

directory = "C:\\Users\\sonjj\\study_spark\\data"
filename = "xAPI-Edu-Data.csv"

lines = sc.textFile("file:///{}\\{}".format(directory, filename))
lines

header = lines.first()
header

datas = lines.filter(lambda row : row != header)
datas

# collect(): 실제 데이터 확인
datas.collect()[:3]

#국적만 추출하기
countries = datas.map(lambda row : row.split(',')[2])
countries

countries.collect()[:3]

#국적 count
result = countries.countByValue()
result

#시각화
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

series = pd.Series(result, name='countries')
series

plt.figure(figsize=(15, 10))
sns.barplot(x=series.index, y=series.values)
plt.show()
```
---

Hadoop
1. HDFS
   1. 파일 시스템(분산 저장)
2. Map Reduce
   1. 연산 엔진
   2. 데이터 집계
   3. Spark의 주 기능
3. Yarn
   1. 리소스 관리
   2. 클러스터 관리

<img src = "CPUFlow.png">

- 컴퓨터 작업 시 HDD/SSD에서 CPU로 데이터가 이동한다.
- 연산에 자주 사용되는 데이터는 위쪽에 저장
- 연산에 자주 사용되지 않는 데이터는 아래쪽에 저장
- HDD/SSD로 갈수록 용량은 크지만 처리 속도가 현저히 느려지기 때문에 데이터를 어디에 저장할지 잘 판단해야 한다.

<img src = "flow.png">

- RAM에서 처리하기 힘든 크기의 데이터는 HDD/SSD와 연동하여 처리
- RAM에서 입부 연산할 데이터를 RAM에 적재, 연산 후 결과를 디스크에 저장
- 단, 속도가 현저히 느려진다

<img src = "Datasplit.png">

- LEADER에서 FOLLOWER을 관리하고, 데이터를 분산하여 전송
- FOLLOWER에서는 LEADER에서 넘겨준 데이터를 받아 실질적인 연산을 처리한다

* Spark에서의 Cluster
  - LEADER역할을 하는 Cluster에서 Dirver Program은 각각의 Worker Node에 연산(Task)을 할당해준다.
  - Worker Node(Follower) Cluster에서는 Executor에서 작업을 수행하고 이를 Cache에 저장한다.
  - Cluster Manager는 어떤 Worker Node에서 Task를 빠르게 수행할 수 있는지 판단하여 분배하는 역할을 한다.



