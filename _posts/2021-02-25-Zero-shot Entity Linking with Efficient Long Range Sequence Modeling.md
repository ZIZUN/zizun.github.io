---
title:  "[엔티티 링킹] Zero-shot Entity Linking with Efficient Long Range Sequence Modeling "
excerpt: "Zero-shot Entity Linking with Efficient Long Range Sequence Modeling   논문 소개"
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
last_modified_at: 2021-02-25T09:06:00-05:00
---
# Introduction

본 논문은 새로 사전학습 없이, 효율적인 포지션 임베딩 방법을 이용해서 긴 범위의 시퀀스 모델링을 하는 방법을 제시하고, 이것을 cross encoder에 적용해 zero shot entity linking SOTA를 달성하는 성과를 보여준다.

![png](/images/el4/1.PNG "그림1"){: width="60%" height="60%"}  

다음 figure는 입력을 mention context(512token) + entity description(512token) 으로 하는 인코더에서 larger position embedding을 적용한 모습이다.


<br>

# Content

제로샷 엔티티링킹에서는 mention context, entity description을 잘 이해하는 능력이 아주 중요하다. 따라서 이 논문에서는 긴 길이의 입력을 받기 위한 방법을 찾기 위 한 시도를 한 것 같다. Logeswaran et al.(2019)에서는 ERLength(입력으로 들어가는 length)를 256으로 설정했지만, 이 논문에서는 더 긴 길이의 입력을 새로운 포지션 임베딩 방법을 이용해 추가적 프리트레인 없이 성능을 끌어올렸다.

버트의 maxlen은 512로 정해져있지만, 이를 늘리기위해 3가지 방법을 시도한다.

![png](/images/el4/2.PNG "그림1"){: width="60%" height="60%"}  



우선 새로운 포지션 임베딩의 size를 1024로 한다고 가정하고 설명하겠다.

그리고 저자는 mention context와 entity description length를 같게 설정했을 때 실험에서 가장 합리적인 결과가 나왔다고 해서 그렇게 했다고 한다.

첫째는, Embeddings_constant 방법인데, 이전 mention context임베딩 의 마지막 포지션의 임베딩 값을 512position 이후의 임베딩에 사용하는 것이다. (두 토큰 사이 거리가 길 경우 상수로 간주하는 경향이 있어 왔다고 한다.)

둘째는, Embeddings_head, 랜덤으로 임베딩을 초기화하는것이다.

셋재는, Embeddings_repeat, 앞부분의 mention context임베딩을 반복하여 그대로 사용하는 것이다.

![png](/images/el4/3.PNG "그림1"){: width="90%" height="90%"}  



다음은 포지션 임베딩 방법에 따른 ERLength별 엔티티링킹 성능을 보여준다. 

repeat방법이 가장 잘나오는 것을 볼 수 있다.

Clark et al.(2019)를 보면 BERT를 어텐션 헤드를 뜯어본 결과 이전, 다음 토큰 즉, local context에 attention이 이루어 진다는 것을 알 수 있고, 따라서 저자는 이를 참고해서 Embeddings_repeat 방법이 mention context와 entity description이 분리되는 경계부분을 제외하고 이런 local 구조를 유지할 수 있게 한다고 가정하고 실험을 진행했다고 한다.

![png](/images/el4/4.PNG "그림1"){: width="90%" height="90%"}  



다음은 ERLength를 Erepeat을 이용하여 늘렸을 때 zeroshot entitylinking 성능변화를 보여준다.

DAP(도메인 적응 사전학습-unsupervised)까지 적용했을 때 우수한 성능을 보여주는 걸 볼 수 있다.

![png](/images/el4/5.PNG "그림1"){: width="90%" height="90%"}  



DLength는 데이터의 길이를 뜻한다.(mention context + entity description)

ERLength는 입력으로 들어가는 길이이다.

이에 따라 Erepeat을 적용한 성능을 보여주는데, 길이가 긴 데이터가 많이 있으면 ERLength를 충분히 능려주는 것이 성능에 도움이 되는 것을 알 수 있다.

DLength가 짧은 데이터에 큰 ERLength를 적용해도 성능이 얼마 안떨어 지는 것으로 보아, 

Erepeat을 적용해 ERLength를 크게 늘리더라 해도 DLength가 짧은 데이터에 대한 성능에 크게 영향을 끼치지 않는 강건한 모습도 보이는 것으로 보인다.


