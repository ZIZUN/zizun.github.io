---
title:  "[엔티티 링킹] Zero-Shot Entity Linking by Reading Entity Descriptions"
excerpt: "Zero-Shot Entity Linking by Reading Entity Descriptions  논문 소개"
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
last_modified_at: 2021-02-24T09:06:00-05:00
---
# Introduction

본 논문은 새롭게 zero shot entity linking task를 정의하는 논문 이다. 그것은 어떤 도메인의 entity에 대한 어떤 labeld 데이터 없이 entity의 description만을 이용해서 학습 때 보지 못한 entity를 링킹하는 것이다. 원하는 도메인의 text는 언어모델 트레이닝에 이용가능하다. (따라서, 언어모델의 언어이해 성능이 중요할 것으로 보인다)

저자는 이 논문의 기여를 세가지로 정리했다.

첫째, 그들은 엔티티링킹의 일반화 성능을 위해, 새롭게 zeroshot 엔티티링킹 테스크를 제안했고 데이터셋도 만들었다.

둘째, mention context와 entity description사이에 이전엔 사용되지 않던 어텐션을 적용했다. 이를 적용하는 것이 이번 테스크에 결정적이다.

셋째, domain adaptive pretraining(DAP)를 적용했다. 그냥 마지막에 해당 task에 맞게 소스도메인데이터(labled)로 supervised learning해주기전에 타겟 도메인에 맞게 프리트레인 하는 것 같다.. (+ 간단하지만 새로운 방법이라고 하는데; 별로 새롭진 않은 것 같다.)


![png](/images/el3/1.PNG "그림1"){: width="60%" height="60%"}  

다음 figure는 제로샷 엔티티링킹에 대해 설명한다. 학습때 전혀 보지않은 엔티티(Orient Expedition)를 test에서 맞추는 모습이다. 다른 resource(e.g alias table, structured data) 없이 entity description만을 이용한다.


<br>

# Content

- Model

모델을 한번 살펴보면
후보 엔티티 생성에는 BM25를 사용했고(등장한 멘션 string으로), 
후보 랭킹엔 멘션 컨택스트+엔티티 description을 입력으로 하는 Full transformer
멘션컨택스트, 엔티티인코딩을 따로하는 pool-transformer
Pool-transformer에 약간의 어텐션을 추가한 cand-pool-transformer 세가지 모델을 사용했다.

- Result

![png](/images/el3/2.PNG "그림1"){: width="60%" height="60%"}  

제로샷 엔티티링킹 baseline 결과들은 다음과 같다.

Full-transformer 구조에 대량의 코퍼스(U_WB)를 학습시키고 source labeled data에 파인튜닝 시킨 결과가 가장 좋은 성능을 나타냈다. 다른 모델보다 잘 나오는 이유는, mention context+entity description을 함게 입력으로 넣어, cross attention이 이루어 지기 때문에(Transformer 구조 특성상)
그렇다. Facebook의 최근 BLINK도 동일한 encoder 구조를 사용한다.


Ganea and hofmann, gupta et al. 의 모델들은 제로샷 성능이 많이 떨어지게 나왔는데 그이유는 이모델들은 labled 된 멘션데이터를 이용하는 설정으로 설계된 모델이기 때문이다.

저자는 말한다. 따라서 논문에서 제시하는 제로샷 엔티티링킹 성능을 올리려면 “언어모델의 reading comprehension 능력”이 매우 중요하다고.


![png](/images/el3/3.PNG "그림1"){: width="90%" height="90%"}  

다음은 domain adaptive pretraining(DAP)를 적용했을 때 나은 결과가 나온다는 실험결과이다.

Source domain labeled data에 파인튜닝 하기 전 마지막 단계에 target domain unlabeld 데이터에 pretrain을 함으로써 타겟 도메인의 성능을 높이는 결과를 가져옴을 확인할 수 있다.

저자는 마지막으로 말한다. 이제 NIL mention, entity detection이나 mention detection, candidate entity generation이 성능향상을 위해 남아있는 주요한 부분이라고.


