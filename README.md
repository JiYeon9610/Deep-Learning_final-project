# Deep-Learning_final-project

2020.11.20 ~ 2020.12.14




# 데이터 설명

## Lobachevsky University Electrocardiography Database
ECG signal database with marked boundaries and peaks of P,T waves and QRS complexes



-200개의 records로 이루어짐

-1개의 record 안에 sex, age(10), dignosis, i, ii, iii, avr, avl, avf, v1, v2, v3, v4, v5, v6

-sex, age(10)의 경우 기본 인적 사항 관련 데이터로 단순 obs1개로 이루어짐

-dignosis는 record 별로 다른 길이의 리스트 형태(진단 결과 관련 정보)

-i, ii, iii, avr, avl, avf, v1, v2, v3, v4, v5, v6은 각각 signal, anno, anno_idx의 dictionary 형태로 이루어짐

--> signal: 모든 record, 모든 record, 유도에 대해 5000개의 리스트로 같은 차원을 가지고 있음

--> anno / anno_idx: record 마다 다른 길이를 가지고 있고, 가끔 오류 데이터 때문에 record 안에서도 그 길이가 다른 경우도 존재. signal 리스트 데이터 중 N, t, p의 위치에 해당되는 인덱스에 대한 정보를 가지고 있는 데이터



# 문제 정의
## 좌심실 비대증 여부 classification 예측 모델 구축하기


### 1. 좌심실 비대증(LHV)이란:

심혈관질환 발생 예측의 중요한 전조질환으로, 심전도 검사의 주된 목적 중 하나.



### 2. 해당 문제 정의의 이유


#### 2-1. 좌심실 비대증 진단의 중요성

2005년 가정의학회지에 기재된 ‘좌심실 비대의 진단 방법으로서 심전도’에 따르면 LVH의 경우, 심전도검사에서 민감도는 낮고 특이도는 높은 경향을 보임. 

즉, 심전도검사를 통하여 LVH로 의심될 확률이 낮다는 것. 질병의 진단 경우, 실제로는 병을 앓고 있지만 그렇지 않다고 진단하는 것은 매우 큰 문제.

LVH를 더 정확하게 진단하는 방법으로는 심장 초음파 검사가 있지만, 비용이 많이 들고 일차의료에서는 누구나 이용할 수 없어 보편화하기 어렵다는 단점.

그렇기 때문에 일차 의료에서 행해질 수 있는 간단한 1차 검사인 심전도 검사 데이터에서 LVH를 잘 분류해내는 것은 빠르게 심장에 이상이 있음을 감지하고 한 발 더 빠른 의학적조치를 가능하게 함.


#### 2-2. 데이터의 분포

보유한 데이터에서 LVH 유무에 대한 분포를 확인해 보면 108대 92. -> 균형 잡힌 데이터 분포

200개의 한정된 데이터를 활용해야한다는 점에서, 불균형한 데이터를 활용할 경우 하나의 클래스로 편향된 분류모델이 될 위험이 큼. 

적은 데이터를 활용하여 높은 예측력을 가진 모델을 구축할 수 있을 것으로 판단하여 해당 문제를 정의하기로 결정.



# 모델링

DNN Framework와 1D-CNN Framework 두 가지를 활용함. 각각의 모델에 활용된 input data가 다름.

## 1. DNN Framework
### Input Data
Anno와 anno_idx는 signal 리스트 데이터 중 각 파형의 위치에 해당하는 인덱스에 대한 정보를 가지고 있는 데이터로, 5,000의 차원을 가지고 있는 시그널데이터에서 핵심적인 정보를 뽑아낼 수 있
는 잠재적 변수라고 판단하였다. 대부분의 anno는 (n)(t)(p)의 순서로 반복되는 양상을 띠고 있는데, 이에 따라 반복되는 (n)(t)(p)의 시그널값들을 평균한 값을 한 record의 시그널 추세를 대변하는 값으
로 활용하기로 하였다. 결론적으로 age, sex 변수는 categorical 변수로 취급하여 one-hot encoding한 변수들로 변환하고, 각 파형의 시작, 피크, 끝의 평균적인 값들을 저장한 9개의 element로 이루어
진 list를 저장해주어 유도 별 9개의 리스트로 이루어진 signal_ntp 변수를 추가해 총 117개의 차원으로 input을 결정하였다.

파형의 특징을 함축한 record_ntp를 LVH 유무에 따라 시각화하여, LVH유무에 따라 파형에 차이가 있음을 확인하였다. 따라서, 이를 통해 5,000개의 시그널을 대신할 새 변수가 LVH 여부를 분류하는데 충분한 학습자료가 될 수 있을 것으로 판단하였다.

### Modeling
앞서 설명한 input 데이터를 활용하여, 입력변수 간의 비선형조합이 가능하며 예측력이 다른 머신러닝 기법들보다 비해 상대적으로 우수한 DNN BInary-Classifier를 모델링 하였다. 

모델을 학습시킬 때, train set과 validation set을 활용하였다. 데이터의 개수가 200개로 매우 적기 때문에 별도의 test set을 만들지 않았다.

모델의 frame으로 3개의 hidden layer를 쌓았으며 hidden layer의 활성 함수로는 +/-가 반복되는 교류에서 –의 흐름을 차단하는 ‘ReLu’를 사용하였다. 마지막 출력층에는 이진 분류를 위한 ‘Sigmoid’ 함수가 사용되었다. 또한, 추가적인 평가 기준 설정을 위한 compile 함수에는 이항 간 분류 문제를 다루는 ‘BinaryCrossEntropy’와, 지그재그 현상이 줄어들고 관성의 효과를 낼 수 있는 momentum의 장점과 최근 값과 이전 값에 각각 가중치를 주어 최근 값을 더 잘 반영할 수 있는 RMSprop의 장점이 적절히 조화된 ‘Adam optimizer’를 사용하였다. 마지막으로 층을 지날수록 weight가 너무 커지거나 작아지는 것을 방지하기 위해 hidden layer사이에 ‘Batch Normalization’을 사용하였다. 또한,  valdation set의 성능이 더 이상 개선의 여지가 없을 때 학습을 종료하는 EarlyStopping callback 함수를 활용하였다. 너무 많은 epoch으로 인해 발생할 수 잇는 과적합 문제를 방지하고, 모델 학습 시간을 효율적으로 절약하기 위함이다. 이를 활용하여 좀 더 개선된 validation accuracy를 확인할 수 있었다.



추가적으로 k-fold cross validation을 활용하였다.

앞선 validation set, train set을 활용한 모델링은 과적합 문제에서 비교적 자유롭지 못하기 때문이다. 고정된 validation set을 활용하였고, 별도의 test set이 없기 때문에 모델의 성능이 나쁘지 않다고 판단 되어도, 이를 실제로 일반화시킬 수 있을지의 여부는 확신할 수 없다. 결과적으로는 모델이 validation set에도 과적합되었을 가능성을 배제할 수 없기 때문이다.

따라서, 모든 데이터를 테스트 세트로 한번씩 활용할 수 있으며, train/validation set에 있는 데이터의 특성으로 인해 발생할 수 잇는 편파된 결과의 가능성을 감소시키는 k-fold cross validation 방법을 활용하였다. 이를 활용하여 추가적인 parameter tuning과 모델링을 진행하였다.

### 한계점
DNN 모델링 결과로, 최종적으로 accuracy 79.6 퍼센트의 모델 성능을 확인하였다.

좋지 않은 결과는 DNN Framework의 한계와 관련이 있다고 판단하였다. 

5000개의 signal 데이터를 9개로 축소하는 과정에서 데이터의 손실이 많이 발생해, 모델의 성능 저하로 이어진 것으로 예상하였다. signal의 데이터가 특정한 주기에 따라 반복되는 시퀀스 데이터(시계열성)인데도 불구하고, 해당 DNN 모델에서는 이러한 주기적인 정보들을 고려하지 못하였기에 추가적으로 정보가 손실된 것으로 판단하였다.


## 2. 1D-CNN Framework
### Input data
 심전도 데이터는 주기적인 파동을 지니고 있는 신호 데이터이기에 평
균적인 값들로 나타낸 변수로는 놓치는 부분이 있으리라 판단하여 5,000개의 시그널값을 전부 사용
하는 또 다른 입력값도 사용할 예정이다. 시그널 값을 전부 사용하는 입력값의 경우, 12개의 유도 값
을 전부 사용하는 것이 아니라 LVH의 진단기준인 Sokolow-Lyon index를 참고하여 V1, V5, V6, aVL 4
가지 유도만을 사용하여 학습의 효율성을 늘리고자 하였다.

### Modeling
앞에서 언급했듯이 핵심적인 특성만 추출한 새로운 변수는 학습에 있어서 놓치는 부분이 있을
수 있다. 이런 상황을 보완하기 위해 4개의 유도의 시그널값을 학습하는 새로운 모델이 필요하였다.
많은 수의 데이터를 효과적으로 학습하기 위해 우리는 1-D CNN을 채택하였다. CNN은 각 레이어의
입출력 데이터의 형상을 유지하고, 필터를 공유 파라미터로 사용하기 때문에 일반 신경망과 비교하여
학습 파라미터가 매우 적다. 또한 많은 연구들이 심전도 데이터 분석을 위해 CNN 모델을 사용하는
것을 확인하였고 그 중 2004년 IOP Conf.Series: Journal of Physics에 기재된 ‘Deep Learning for ECG
Classification’을 참고하여 1D-CNN을 선정하였다. 1D-CNN은 고정적인 길이의 특징을 뽑아내는데
특화되어 있으며, 위치에 덜 민감한 정보를 잘 처리한다. 우리가 학습할 데이터의 경우 시간에 따라
5,000개의 심전도 전압으로 고정되어 있으며, 전압값이 중요하므로 위치에 덜 민감한 1D-CNN이
적당하다고 판단하였다. 총 5개의 hidden layer를 쌓았으며, CNN 함수의 커널 사이즈는 (250, 4),
strides는 (10, 1)로 설정하였다. 그 이유는 보통 사람의 심박 수가 1분에 60~120 사이에 위치한다는
것과 초당 500개의 전압을 측정한다는 것을 고려하여, 놓치는 정보를 최대한 줄이기 위해서이다.
또한 우리는 차원의 수가 줄어들기를 원하기 때문에 valid Padding으로 데이터 차원의 축소를 막는
padding을 사용하지 않았으며, 위의 DNN과 같은 이유로 ‘Batch normalization’과 ‘Relu’함수를 이용해
출력값을 조절하였다. 추가로 시그널은 10초 동안 반복되기 때문에 중복된 특징이 많아 층마다 0.5의
비율로 ‘drop out’을 사용하여 입력 벡터를 줄였다. 이 CNN 또한 최종적으로 이진 분류모델이기에
출력층의 활성 함수와 model.compile함수의 파라미터들은 DNN과 똑같이 구성하였다.

