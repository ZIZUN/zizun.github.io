---
title:  "[엔티티 링킹] End-to-End Neural Entity Linking"
excerpt: "End-to-End Neural Entity Linking 논문 요약"
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
last_modified_at: 2020-08-30T08:06:00-05:00
---
# Abstract

Entity linking = 문장에서 개체mention을 찾아내서 지식베이스 (e.g. wikipedia)에 연결하는 자연어처리 Task
![png](/images/table1.png "표1"){: width="100%" height="100%"}  

1. 파이프라인 방식 : Mention Detection, Entity disambiguation 따로
2. End to End model : MD, ED 결합한 모델

![png](/images/image1.png "그림1"){: width="100%" height="100%"}  

## 1. Embedding

Character 임베딩(같은 가중치를 공유하는 하나의 모델 사용) 과 
Word2vec을 통해 사전학습된 단어임베딩을 concatenate해서 하나의 word embedding을 생성(V_k),

Bi-LSTM을 통해 context를 이해한 임베딩을 생성 (X_k)

Entity embedding의 경우는 따로 word-entity 의 분포에 따라 사전학습 된 것을 쓴다.<br>
Deep Joint Entity Disambiguation with Local Neural Attention 논문을 참고.<br>
나도 잘 모르겠다. 아시는 분은 댓글 남겨주세요.

## 2. Mention Representation

멘션의 첫 단어, 끝 단어의 임베딩, “Soft head 임베딩” 을 Concatenation 하여 고정된 크기의 멘션 임베딩 생성(X_m)

![png](/images/image2.png "그림2"){: width="100%" height="100%"}  

## 3. Final local Score

멘션임베딩과 후보엔티티 임베딩이랑 dot product하여 유사도를 구하고, 로그 사전확률과 결합하여 레이어를 한번 거쳐 context를 이해한 final local score를 얻는다.<br>
(이 부분의 모델을 long range context attention이라 부른다)

![png](/images/image3.png "그림3"){: width="100%" height="100%"}  

## 4. Training

gold mention-entity pair코퍼스(정답)를 가졌다고 가정하고, Input 문서에 대해 가능한 span을 다 구한다.<br>
그 모든 span에 대해 모두 학습 -> “all spans training”<br>
ED만 필요할 경우 gold pair일 경우만 학습. ->”gold spans training”

![png](/images/image4.png "그림4"){: width="100%" height="100%"}  

## 5. Inference

EL의경우 모든 가능한 Span, ED의경우 gold pair에 맞는 Span을 뽑는다.<br>
Score 한계점 δ보다 Ψ 가 높을경우 그 span에 entity를 linking함. span이 겹치지 않도록 해야된다.

## 6. Global Disambiguation

“Local score”만으로는 입력하는 문서내의 span끼리의 연관관계를 반영하기 부족.<br>
따라서 global disambiguation방법을 넣었는데, Local score가 충분히 높은 mention span, entity 조합 V_G의 모든 entity들의 평균(자신의 mention은 제외) 과 자신의 엔티티와 유사도를 구한 값을 Global score로 쓴다.

![png](/images/image5.png "그림5"){: width="100%" height="100%"}  