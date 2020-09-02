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
# Introduction

Entity linking = 문장에서 개체mention을 찾아내서 지식베이스 (e.g. wikipedia)에 연결하는 자연어처리 Task
![png](/images/table1.png "표1"){: width="100%" height="100%"}  

1. 파이프라인 방식 : Mention Detection, Entity disambiguation 따로
2. End to End model : MD, ED 결합한 모델

이번 소개드릴 논문에 나오는 모델은 EndtoEnd모델이다.

MD, ED 결합한 Endtoend방식 최초의 모델이고, 따라서 Mention Detection용 모델을 따로 사용하지 않으므로 가능한 Mention span을 모두 고려한다.

하지만 train set과 다른 annotation conventions를 따르는 test set에 대해 평가할 경우,<br>
EndtoEnd방식이 아닌 논문에서 제시한 NER + ED model을 쓰는것이 낫다고 말한다.


![png](/images/image1.png "그림1"){: width="100%" height="100%"}  

## 1. Embedding

Character 임베딩(같은 가중치를 공유하는 하나의 모델 사용) 과 
Word2vec을 통해 사전학습된 단어임베딩을 concatenate해서 하나의 word embedding을 생성(V_k),

Bi-LSTM을 통해 context를 이해한 임베딩을 생성 (X_k)

Entity embedding의 경우는 따로 word-entity 의 분포에 따라 사전학습 된 것을 쓴다.<br>
자세한 것은 Deep Joint Entity Disambiguation with Local Neural Attention 논문을 참고.<br>
이 부분은 잘 설명을 못하겠다.

## 2. Mention Representation

멘션의 첫 단어, 끝 단어의 임베딩, “Soft head 임베딩” 을 Concatenation 하여 고정된 크기의 멘션 임베딩 생성(X_m)<br>
멘션 span의 크기는 거의 2보다 작거나 같기 때문에 "Soft head 임베딩"은 성능향상에 크게 의미가 없다.

![png](/images/image2.PNG "그림2"){: width="100%" height="100%"}  

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


# Experiments

학습에는 AIDA/CoNLL 데이터셋(946개의 문서와 문서 내의 18448개의 mention)을 사용했고, Test에는 ED, EL를 테스트 하기 위해 만들어진 Gerbil Platform을 이용했다.

“strong matching” : 멘션의 바운더리가 정확하고, gold(정답) entity annotation을 예측<br>
“weak matching” : 멘션의 바운더리가 정확하지 않고 겹쳐치고, gold entity annotation을 예측


![png](/images/image6.png "그림6"){: width="100%" height="100%"}  
<Entity Linking, Gerbil 플랫폼에서 “strong matching” 결과>

빨간색이 first 파랑색이 second score 라고 보면 된다.<br>
논문에서 말했다시피, AIDA셋으로 train해서 AIDA셋에서 test했을 때 EL모델이 좋은 성능을 내는 것을 볼 수 있고, 
train set과 다른 annotation conventions를 따르는 test set에 대해 평가할 경우, 
NER + ED model이 좋은 성능을 내는 것을 확인할 수 있다.

![png](/images/image7.png "그림7"){: width="100%" height="100%"}  
<Entity Linking, Gerbil 플랫폼에서 “weak matching” 결과>

"Strong matching"보단 score가 미세하게 오른걸 볼 수 있다.

![png](/images/image8.png "그림8"){: width="100%" height="100%"}
<AIDA A 데이터셋에서 멘션 길이 별 맞는 entity 예측할 확률>

데이터셋을 보면 멘션길이가 1~2가 90프로 이상을 차지하는 것을 볼 수 있다.<br>
따라서 "soft head embedding"이 왜 성능을 올리는데 크게 의미 없음을 알 수 있다.<br>
<br>
윗값은 정답 entity를 맞출 확률이고, 아랫값은 정답 entity를 맞춘것 + 
후보엔티티에서 score가 가장 높은 엔티티가 정답엔티티 이지만 score가 "한계점"을 넘지 못한 것을 포함한 확률이라 보면 된다.
따라서 아랫값이 조금 더 높다.<br>
그리고 멘션의 길이가 길어질수록 성능이 현저히 떨어지는 것을 보인다.

![png](/images/image9.png "그림9")
![png](/images/image8.png "그림8"){: width="100%" height="100%"}

![png](/images/image9.png "그림9")
![png](/images/image8.png "그림8"){: width="100%" height="100%"}
![test3](/images/image9.png)
<AIDA A 데이터셋에서 멘션 길이 별 맞는 entity 예측할 확률>

빨강(1,3,5) = 멘션 잘 찾고 엔티티도 잘 찾았는데, high score를 못받아서 not annotation<br>
주황(3,4) = Korean War을 골라야 되는데 Korean - > korea(entity)로 해버렸다.

# Reference

  * [EndtoEnd Neural Entity Linking][1]

[1]:https://arxiv.org/pdf/1808.07699.pdf

