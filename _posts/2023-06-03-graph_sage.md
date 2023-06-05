---
title:  Inductive Representation Learning on Large Graphs (NIPS 2017)

excerpt: GNN, Graphsage , NIPS

toc : true
toc_sticky : true  

use_math: true

categories:
  - papers
tags:
  - papers
  - gnn
  - graphsage
---

# 0. Abstract

large 그래프에서 노드들에 대한 저차원 임베딩은 다양한 예측 테스트에 몹시 효과적이라는 것이 증명되었다. 하지만 지금까지의 방법들은 그래프에서의 모든 노드가 임베딩 과정에서 고려되어야 한다. 이러한 방법들은 
근본적으로 [transductive](https://en.wikipedia.org/wiki/Transduction_(machine_learning)) 하며 보지 못했던 새로운 노드에 대해서까지는 일반화하지 못한다. 이제
본 논문에서는 inductive한 방법의 프레임워크인 GraphSAGE 를 제안하는데, 이전에 보지 못한 데이터에
대해서도 효과적으로 임베딩을 생성할 수 있도록 노드의 피쳐 정보들을 활용한다. 모든 노드의 각자의 임베딩을 훈련하는 대신에, 
본 논문에서는 한 노드의 근처의 이웃들의 피쳐를 샘플링하고 집계하여
임베딩을 생성하는 함수 그 자체를 학습한다. 

# 1. Introduction 

![fig1](https://user-images.githubusercontent.com/113276452/243048496-7a3f6e99-44d0-4af9-8672-872bb7b41fcc.png)

large 그래프에서의 노드에 대한 저차원 임베딩은 다양한 분야의 예측과 그래프 분석 테스크에서 사용하는
피쳐로 몹시 효과적이라는 사실이 증명되었다. 기본적인 아이디어는 노드의 그래프에서에 이웃들에 대한 고차원 정보를 
dense 한 저차원 임베딩으로 퍼트리는 차원 감소 테크닉들을 사용하는 것이다. 
이 노드 임베딩들은 다운스트림 머신 러닝 테스크들에 이용되고 노드 분류 , 노드 클러스터링, 링크 예측과 같은
테스크들에 도움을 준다.

그러나, 선행 연구들은 하나의 정해진 그래프로의 노드 임베딩에만 초점을 맞추었다. 이 연구들을 많은 
현실 사례에 적용하려면 실제로는 임베딩이 새롭게 입력되는 노드들 혹은 전체(부분) 그래프에 대해서
빠르게 생성되어야 한다. 

이러한 inductive 한 기능은 계속 진화하고 끊임없이 새로운 노드
(예시 :유튜브 사용자들과 영상들)들을 만나는 환경에서 작동하고 있는 
초고효율의 생산 머신 러닝 시스템에는 필수적이다. 
노드 임베딩을 inductive 하게 생성하는 접근방식은 한편 동일한 형태의 피쳐들을 가지고 
이것을 여러 그래프들에 일반화하는 것을 가능하게 한다. 

Inductive 한 노드 임베딩 문제를 푸는 것은 매우 어렵다. transductive 한 문제 상황과 비교하였을 때, 
새로운 노드들에 대한 일반화는 새로 관찰된 부분 그래프들을 이미 알고리즘에
의해 최적화된 임베딩에 **"맞추는(aligning)"** 것을 요구한다. inductive 한 프레임워크는 
노드의 local 한 역할, global 한 위치를 보여주는 노드의 이웃에 대한 구조적인 성질들을 인식하는 방법을 학습해야만 한다.

노드 임베딩을 만들어내는 지금까지의 접근들 대부분이 근본적으로 transductive 하였다.
이 방법들은 하나의 이미 주어진 그래프에 있는 노드에 대해서만 예측을 수행하기 때문에,
새로 들어오는 데이터에 대해서까지 일반화 할 수가 없었다. 이런 접근들도 inductive 한
문제 세팅으로 바꿀 수 있지만, 많은 컴퓨팅 리소스와 추가적인 경사하강법 단계들을 요구한다.

최근에는 콘볼루션 연산을 사용한 임베딩 방법도 있다(GCN) GCN 은 transductive 한
세팅에서만 사용되고 있지만 이 논문에서 GCN 을 inductive 한 비지도 학습으로까지
확장하고, 단순한 콘볼루션이 아니라, 학습 가능한 집계 함수를 사용하는 프레임워크로 일반화 할 것이다.

이 논문에서 제안하는 GraphSAGE (Sample and aggreGatE) 는 inductive 한 노드 임베딩을 위한
보편적인 프레임워크이다. 이 논문에서는 노드 피쳐를 새로 입력되는 노드들에 대해 일반화할 수 있는 임베딩
함수를 학습하기 위해 사용한다. 노드 피쳐를 학습 알고리즘에 통합하여 각 노드의 이웃에 대한 위상수학적
구조와 이웃들에서의 노드 피쳐의 분포를 동시에 학습할 수 있게 된다. 이 접근방식은 또한 노드 피쳐가 풍부한
그래프뿐만 아니라 피쳐가 없는 그래프에 대해서도 적용가능하다.

각 노드에 대한 분리된 임베딩을 훈련하는 대신에, 집계 함수들의 집합을 훈련한다. 그렇게 함으로써 
노드의 local 이웃들로부터 피쳐 정보를 집계하는 방법을 학습한다.(Figure 1 을 보자.)
집계 함수들은 주어진 노드들로부터 일정한 거리 또는 탐색 깊이로부터 떨어진 정보들을 집계한다.
추론(테스트) 단계에서, 학습된 집계 함수들을 사용하여 새로운 노드들에 대한 임베딩을 생성한다.
비지도 손실 함수를 GraphSAGE 가 테스크에 특정한 지도 없이도 훈련 가능할 수 있도록 하였다.
한편 GraphSAGE 가 완전한 지도학습 방법으로만 훈련되는 것도 보였다. 

# 2. Related work

(생략)

# 3. Proposed method : GraphSAGE

핵심 아이디어는 로컬한 이웃으로부터 피쳐 정보를 집계하는 방법을 학습한다는 것이다.

3.1 에서는 모형 파라미터가 이미 학습되었다고 가정하고, GraphSAGE 의 임베딩 생성 알고리즘을
설명할 것이다.

3.2 에서는 일반적인 그레디언트 경사하강법과 역전파 테크닉을 이용하여 어떻게 모형 파라미터를
학습하는 지 설명할 것이다.

## 3.1 Embedding generation algorithm (forward propagation)

![](https://user-images.githubusercontent.com/113276452/243048562-4764034b-72cc-4b18-be29-cdcf1a042ae8.png)

입력 : 그래프 : $$\mathcal{G(V, E)}$$ , 피쳐 $$\{x_{\mathcal{v}}\}$$ ,
깊이(레이어 갯수, 탐색 거리) K, 가중치 행렬 $$W^{k}$$, 비선형 함수 $$\sigma$$ , 
미분가능한 집계 함수, $$\text{AGGREGATE}_k, k \forall \in \{1,2 \cdots, K\}$$, 이웃 $$\mathcal{N}: v \rightarrow 2^{\mathcal{V}}$$.

출력 : 각 노드에 대한 최종 임베딩 벡터 $$\mathbf{Z}_v$$.

> **알고리즘 설명** 

1. $$h_{v}^0 = x_v$$.
2. 바깥 루프 : 레이어 갯수 K 만큼 반복 , 안쪽 루프의 결과 임베딩의 크기를 Normalize 한다.
3. 안쪽 루프 : 고정된 레이어 k에 대하여 그래프의 모든 노드에 대해 반복 <br> - 고정된 노드가 주어지면 그  노드 이웃들의 정보를 함수를 통해 집계함. <br> - 이전 시점 노드의 임베딩과 집계한 이웃의 정보를 Concat 하고 완전 FC Layer 통해 비선형함수에 통과시킴.

이 알고리즘은 현재 Full-Batch 이다. 이를 Mini-batch 까지 확장하려면 입력된 노드에 대해서
그 노드의 이웃을 샘플링해야한다 이 경우 안쪽 루프를 반복할 때, 모든 노드에 대해서 진행하지는 않는다.
(자세한 부분은 Appendix 에 있다.)

> **Weisfeiler-Lehman Isomorphism Test**

WL Test는 서로 다른 두 그래프가 isomorphic(동일한 그래프) 인 지를 판별하는 알고리즘 이라 한다.
논문에서는 GraphSAGE 가 이 알고리즘의 한 경우에 대한 신경망을 이용한 근사(approximation)라고 설명하고 있다.

참고로 WL Test 를 통과한다고 두 그래프가 항상 isomorphic 인 것은 아니라고 한다.
(만약 통과하지 못한다면, isomorphic 일 수는 없다.)

> **Neighboorhood definition**

이 논문에서는 고정된 사이즈의 이웃 집합을 각 반복 k 마다 다르게, uniform 하게 샘플링하였다.
이렇게 하지 않으면 시간복잡도가 $$O(\lVert \mathcal{V} \rVert)$$ 가 된다.
GraphSAGE 에서의 시간복잡도는 $$O(\lVert \prod_{i=1}^{K} S_i)$$ 가 된다.
여기서 $$S_i$$ 와 K 는 하이퍼 파라미터이다. 실험에서 K =2 , $$S_1 \cdot S_2 \leq 500$$
이 가장 좋은 성능을 보였다고 한다. 


## 3.2 Learning the parameters of GraphSAGE
비지도학습 상황에서 최종 임베딩 벡터 $$\mathbf{z_u], u \in \mathcal{V}$$ 에 대한 손실 함수를 정의해보자.
이 손실 함수는 유사한 임베딩을 갖는 노드끼리는 더 가깝게, 서로 다른 임베딩을 갖는 노드끼리는 더 멀게 만들도록
집게 함수와 가중치 벡터들의 파리미터들을 학습시킨다. 그 형태는 다음과 같다.

![](https://user-images.githubusercontent.com/113276452/243154087-8b8d378a-0cff-4f8f-873e-d2d04665f99c.png)

이때 ,

- $$v$$ : 고정된 길이의 랜덤 워크에서 노드 $$u$$ 근처에서 같이 발생하는 노드 
- $$\sigma$$ : 시그모이드 함수
- $$P_n$$ : negative 샘플링 분포 (이웃에 포함되지 않는 노드를 의미함)
- $$Q$$ : negative 샘플의 갯수 
- $$z_u$$ : 손실 함수에 입력되는 임베딩

논문에선 $$z_u$$ 가 $$u$$의 로컬한 이웃에서 수집된 피쳐들로부터 생성되었음을 강조하고 있다 <br>
이는 look-up 조회하는 방법으로 각 노드 마다의 임베딩을 고유하게 학습시킨 것과는 구분된다.   

비지도 학습 손실함수는 주어진 다운스트림 테스크에 따라 적절한 다른 손실함수(예 : CE Loss) 로 대체할 수 있다.



## 3.3 Aggregator Architectures
그래프에서 노드의 이웃들은 순서를 가지지 않는다. <br>
그래서 집계 함수는 순서가 없는 벡터 집합에 대한 연산을 해야만 한다. <br>
이상적으로는 집계 함수는 대칭적이어야 한다(입력에 순서에 무관하다는 의미이다.) 그러면서도 학습 가능해야 하며
representational capacity 를 높게 가져가야 한다. 집계 함수의 대칭성은 신경망이 임의의 순서를 가진
노드 이웃의 피쳐 집합에 대해서 적용 가능하다는 의미이다. 

이 논문에서는 이러한 집계 함수의 후보로 3가지를 사용한다.

- Mean <br> 평균을 취하는 것은 transductive 한 GCN 의 콘볼루션 전파 법칙과 거의 동일하다. <br> 특히, 알고리즘 1 의 
4번과 5번 lines 를 다음 식으로 대체함으로써 GCN의 inductive 한 변형을 이끌어낼 수 있다.
![](https://user-images.githubusercontent.com/113276452/243164268-41fbf641-663a-49d6-bdda-eb76ed2f58b7.png)
이러한 convolutional aggregator 이 다른 집계 함수의 후보와 구분되는 점은 노드의 이전 레이어의 임베딩과 노드의 이웃 간의 
concatenation 을 실행하지 않는다는 점이다.

- LSTM <br> LSTM 은 Mean 함수와 비교할 때, 풍부한 expressive capability 를 지닌다. <br> 그런데
LSTM 은 순서 무관한 연산을 수행하지는 않는다.(LSTM 은 입력을 순차적으로 받는다) 이 논문에서는 LSTM을
노드의 이웃에 대해 임의의 순열에 적용하여 순서 무관한 집합에 대한 연산을 달성하였다. 

- Pooling <br> max pooling 경우를 보자.
![](https://user-images.githubusercontent.com/113276452/243165383-c52daa82-d925-4012-89f5-a76816a00045.png)
이웃 집합에 element-wise pooling 이 적용된다. $$\sigma$$ 는 임의의 비선형 함수이다. pooling 전에
적용되는 함수는 여러 층의 퍼셉트론 신경망일 수 있지만, 이 실험에서는 하나의 층만을 사용하였다. 


# 4. Experiments

![](https://user-images.githubusercontent.com/113276452/243165501-5b9afae3-0e75-4dd5-872b-12aca5dd1433.png)


3개의 데이터셋에 대하여 지도 학습에서의 손실 함수를 같이 사용한 경우와 그렇지 않은 경우의 <br> F1-score 를 베이스라인 모형들과 비교하였다.

# 5. Theoretical analysis

![](https://user-images.githubusercontent.com/113276452/243165450-13743c3c-7a82-4948-a90a-f10770a4d5fb.png)

이 Theorem 은 특정 조건을 만족하면 그래프의 모든 노드의 임베딩을 특정 클러스터링 계수에 가깝게 근사할 수 있다는 것이다.
(클러스터링 계수는 그래프에서 노드의 바로 옆 이웃이 얼마나 잘 모였는지 측정하는 measure 라고 한다.)
그래서 이 이론을 통해 GraphSAGE 가 그래프 구조를 학습할 수 있다는 것을 보여준다.

Theorem 의 증명의 기본 아이디어는 각 노드가 유니크한 피쳐 representation 을 가지고 있다는 사실이다. 

이 증명은 pooling 집계함수의 성질에 의존하는데, 이를 통해 GraphSAGE-pool 모형이 GraphSAGE-mean 보다
성능이 좋다는 것을 짐작할 수 있었다고 한다.

# Appendices 
![](https://user-images.githubusercontent.com/113276452/243181218-53d294fa-a1f5-4ed9-8f29-bcae3f1497d5.png)

알고리즘 2는 알고리즘 1의 minibatch 버전이다.

핵심 아이디어는 노드들을 샘플링하는 것이다. 

Lines 2~7 이 샘플링하는 단계인데 집합 $$\mathcal{B}^k$$ 는 $$v \in \mathcal{B}^{k+1}$$ 노드의
임베딩을 계산할 때 필요한 노드들을 가지고 있다. 그 노드들을 다음 배치에 넘겨준 뒤, 다시 노드들의 이웃을 샘플링한다. 
나머지 단계는 알고리즘 1과 동일하다.