---
title:  "[엔티티 링킹] Fine-Grained Entity Typing for Domain Independent Entity Linking"
excerpt: "Fine-Grained Entity Typing for Domain Independent Entity Linking 논문 요약"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/logo.jpg

categories:
  - Entity Linking
tags:
  - Entity Linking
  - Entity Disambiguation
  - Entity Typing
  - 개체 연결
  - 중의성 해소
last_modified_at: 2020-11-01T08:06:00-05:00
---
# Introduction

이번에 소개할 논문은 엔티티 타이핑을 이용해 학습에 사용된 특정 도메인을 제외하고, 다른 도메인에도 독립적으로 잘 작동하는 엔티티링킹 모델을 소개하는 논문이다.

실제로 실험결과를 보면 다른 Domain-independent Entity linking system보다 좋은 성능을 나타낸다고한다.

저자는 논문에서 다루는 내용을 크게 이렇게 소개했다.
1. 엔티티링킹을 순수 엔티티 타이핑 으로 풀어낸다.
2. 위키피디아의 카테고리들을 바탕으로 엔티티 타이핑 데이터셋을 구축 한후 엔티티 타이핑 모델을 학습시킨다.
3. 이 학습된 우리 모델을 두개의 도메인 특화 데이터셋에 eval해보고, 그 도메인 데이터셋으로 학습된 모델보다 우리 모델이 더 좋은 성능을 내는 것을 보인다.

하나하나 내용을 찬찬히 살펴보도록 하자.

우선 왜 엔티티 타이핑을 엔티티 링킹에 이용하는가??

저자는 다음과 같이 예를 들었다.

Wikipedia dump에서 Ant라는 mention은 96퍼센트의 insect_Ant, 0.8퍼센트의 Apche_Ant로 나타난다.
따라서 Apache_Ant는 아주 희귀한 엔티티이기 때문에, 넓은 도메인에서 학습된 모델은 insect_Ant를 더 많이 학습하고 선호할 것으로 보인다.

하지만, 엔티티 타이핑을 이용하면 이 문제가 많이 적어지게된다. Apache_Ant의 카테고리로는 Software가 있을 수 있는데, 
이 Software는 mention이 나타난 문맥에서 더 쉽게 예측할 수 있을거라 보여지기 때문이다.(연관되어)

그리고 만약 "Ant"라는 멘션의 카테고리로 software를 찾게 된다면 다른 연관 정보들 필요없이 insect_Ant가 아닌 Apache_Ant로 예측하여 disambiguation 할 수 있을 것이다.

따라서 희귀한 엔티티 (eg. Apache_Ant)를 직접 학습하는 것보다, 엔티티의 카테고리 정보들을 predict하는 것이 더 효율적이라고 저자는 생각했다.

즉, entity로 그냥 매핑하는 것보다 category로 매핑하는 모델을 만드는 것이 일반화 성능에도 좋고, 모델 성능에도 좋을 것임을 대충 예감할 수 있다 ㅎ

# Setup

엔티티링킹을 위한 standard한 데이터셋은

D_el = {(mention, context, entity, a set of cadidate entitys) ... }

다음과 같이 구성되어 train/dev/test 로 쪼개어 학습이 이루어진다.

하지만 우리 모델에는

D_wiki = {(mention, context, a set of Wikipedia categories) ... }

다음과 같이 구성된다. 

Φ : (mention, context) → T

따라서 mention과 context로 category를 예측하는 entity typing 모델을 훈련시킨다고 볼 수 있다.

e = Ω(Φ(m, c), a set of cadidate entitys).

entity typing모델에서 나온 score값과 후보 엔티티들의 카테고리들과 비교해서 score를 매겨 entity를 prediction한다.

학습된 모델로 D_el의 dev/test셋에 대해 eval 해보며 standard entity linking 모델과 성능을 비교해봄으로써 성능을 체크한다.

우리 모델은 entity를 예측하는 것이 아니라(entity 자체의 정보는 이용안함으로써) 결국 category를 예측 하는 것이기 때문에
 wikipedia train set을 단순히 기억하는 것보다 좋은 일반화 성능을 얻었다고 저자는 말한다.

# Model

모델에 대해 자세히 살펴보면,

우선 ELMO를 이용해 sequences of contextualized word vectors c(context), m(mention)을 만들어낸다.

C = Attention(bi-LSTM([c; p]))

얻어낸 c와 p(location embedding)을 concat하여 bilsm attention layer를 통해 representation C를 얻는다.

M_word = Attention(bi-LSTM(m)).

마찬가지로 mention도 같은 과정을 거쳐 representation을 얻고,

M_char = 1D_CNN(m_char)

1D 컨볼루션을 통해 mention의 character-level representation을 얻어낸다.

V = C ; M_word ; M_char ∈ R_d

이를 모두 concat해서 d차원의 V를 얻어낸다.

이 V를 위한 (카테고리 갯수 X d차원)의 weight 매트릭스 W를 만들고

T = o(WV)

W와 V의 행렬곱 결과에 element-wise Sigmoid를 취해 마지막 결과인 probability vector T를 얻는다.

# Train

[그림]

학습은 다음과 같이 간단한 binary crossentropy 식으로 loss함수를 구성해 이루어짐을 알 수 있다.

i는 각 카테고리들을 가르키고, y는 mention이 이 카테고리를 포함한다면 1 아니면 0, t는 모델의 추론값이라고 보면 된다.

# Inference






