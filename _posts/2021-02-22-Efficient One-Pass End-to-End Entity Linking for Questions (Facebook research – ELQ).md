---
title:  "[엔티티 링킹] Efficient One-Pass End-to-End Entity Linking for Questions (Facebook research – ELQ)"
excerpt: "Efficient One-Pass End-to-End Entity Linking for Questions (Facebook research – ELQ)  논문 소개"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/logo.jpg

categories:
  - Entity Linking
tags:
  - Entity Linking
  - Entity Disambiguation
  - 개체 연결
  - 중의성 해소
last_modified_at: 2021-02-22T08:06:00-05:00
---
# Introduction

본 논문은 biencoder구조를 이용한 end-to-end entity linking model을 소개한다. Facebook의 이전 모델은 2stage(biencoder + cross encoder)로 이루어졌었지만, 이번엔 biencoder만을 이용한다 성능 보단 효율, 속도에 초점을 두었다. entity encoding은 file로 미리 저장하여 search에 이용함으로, 하나의 인코더(BERT)만 forward를 하면 된다. One pass구조로 이루어져있어서 상당히 효율적이고 빠르다.

엔티티링킹으로 얻은 엔티티는 wikidata, Wikipedia 항목에 1대1 mapping된다고 볼 수 있다. 따라서, wikidata(KB)를 이용해서 knowledge graph를 형성하고 GCN, GNN 등을 이용해 그래프에서 representation을 뽑아 answer이 있는 article위치, answer span위치를 찾는 방법으로 open domain QA에 활용 될 수 있다. 따라서 본 논문에서 제시한 엔티티링킹 모델을 QA에 적용하여 성능을 향상 시킨 점도 주목할 만하다.


![png](/images/el2/1.PNG "그림1"){: width="100%" height="100%"}  

<br>


# Content

모델 구조를 조금 구체적인 수식으로 보면,

- Mention Detection

![png](/images/el2/2.PNG "그림1"){: width="70%" height="70%"}  


멘션에 대한 probability를 다음과 같이 구한다. I,j는 멘션의 시작과 끝의 인덱스라고 보면 되고, S(i)는 입력에서 i인덱스에 해당하는 토큰을 입력으로 받는 linear layer라 보면 된다.

- Entity Disambiguation

![png](/images/el2/3.PNG "그림1"){: width="70%" height="70%"}  

q 는 mention의 한 token에 대한 representation이라 보면되고 이를 모두 sum해서 entity encoding과 내적하여 score를 구한다.

![png](/images/el2/4.PNG "그림1"){: width="70%" height="70%"}  

Wikipedia의 모든 엔티티에 대해 스코어를 구하고, 해당 entity가 정답일 probability를 구한다.


- End to End Loss

![png](/images/el2/5.PNG "그림1"){: width="70%" height="70%"}  

Mention detection loss는 다음과 같이 구성된다. Binary Cross entropy로 구성했다.

![png](/images/el2/6.PNG "그림1"){: width="70%" height="70%"}  

Entity disambiguation loss 는 다음과 같이 구성된다.

그리고 이 두가지 로스를 합쳐서 학습을 진행한다.

전 논문과 마찬가지로 hard negative mining을 적용한다.

In contrast to a two-stage pipeline which first extracts mentions, then disambiguates entities (Fevry et al. ´ , 2020), a joint approach grants us the flexibility to consider multiple possible candidate mentions for entity linking.

논문에서 인용한 건데, 잘 설명한 것 같아서 가져왔다. Two stage로 구성한 것보다 더 유연하게 여러 개의 후보 멘션을 고려하는게 가능하단 것이다.

![png](/images/el2/7.PNG "그림1"){: width="100%" height="100%"}  

실험 결과인데, Training data – Wikipedia, webqsp 항목은 각 데이터 로만 train 한 것이다. Webq네는 총 100epoch, wikipedia는 1epoch 돌렸다고 보면 된다.
Wikipedia + webqsp 는 wikipedia로 한번 학습하고, webqsp로 파인튜닝 한 것이다.
다양한 목적의 링킹이라면, wikipedia로만 학습시킨 것이, 어떤 원하는 도메인(e.g. QA)에 대한 엔티티링커를 만드는 것이 목적이라면 맞는 데이터셋에 파인튜닝 하는 것이 더 좋은 성능을 냄을 알 수 있다.

![png](/images/el2/8.PNG "그림1"){: width="100%" height="100%"}  

다음은 엔티티링킹 시스템을 지식그래프기반 QA시스템에 적용한 실험결과이다. Webqsp로 파인튜닝했을때 wepqsp데이터셋에 대한 test성능은 잘나오는 걸 알 수 있지만, 다른 qa 데이터셋에 대해서는 오히려 Wikipedia 데이터셋으로만 학습시킨 링커를 이용했을 때 오히려 성능이 잘 나온걸 알 수 있다. 

따라서 QA성능을 올리기 위해서 엔티티링커를 택할 때, 무조건적으로 qa데이터에 파인튜닝 된 링커를 이용하는 것보단 qa내에서도 도메인에 맞는 데이터로 링커를 finetuning해서 이용하는 것이 정답이지 않을까 싶다.
