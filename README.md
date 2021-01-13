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


1. 좌심실 비대증(LHV)이란:
심혈관질환 발생 예측의 중요한 전조질환으로, 심전도 검사의 주된 목적 중 하나.


2. 해당 문제 정의의 이유

2-1. 좌심실 비대증 진단의 중요성
2005년 가정의학회지에 기재된 ‘좌심실 비대의 진단 방법으로서 심전도’에 따르면 LVH의 경우, 심전도검사에서 민감도는 낮고 특이도는 높은 경향을 보임. 
즉, 심전도검사를 통하여 LVH로 의심될 확률이 낮다는 것. 질병의 진단 경우, 실제로는 병을 앓고 있지만 그렇지 않다고 진단하는 것은 매우 큰 문제.
LVH를 더 정확하게 진단하는 방법으로는 심장 초음파 검사가 있지만, 비용이 많이 들고 일차의료에서는 누구나 이용할 수 없어 보편화하기 어렵다는 단점.
그렇기 때문에 일차 의료에서 행해질 수 있는 간단한 1차 검사인 심전도 검사 데이터에서 LVH를 잘 분류해내는 것은 빠르게 심장에 이상이 있음을 감지하고 한 발 더 빠른 의학적조치를 가능하게 함.

2-2. 데이터의 분포
보유한 데이터에서 LVH 유무에 대한 분포를 확인해 보면 108대 92. -> 균형 잡힌 데이터 분포
200개의 한정된 데이터를 활용해야한다는 점에서, 불균형한 데이터를 활용할 경우 하나의 클래스로 편향된 분류모델이 될 위험이 큼. 
적은 데이터를 활용하여 높은 예측력을 가진 모델을 구축할 수 있을 것으로 판단하여 해당 문제를 정의하기로 결정.


