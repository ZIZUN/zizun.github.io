---
title:  "[엔티티 링킹] Scalable Zero-shot Entity Linking with Dense Entity Retrieval (Facebook research – BLINK)"
excerpt: "Scalable Zero-shot Entity Linking with Dense Entity Retrieval (Facebook research – BLINK)  논문 소개"
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

본 논문에서 제시하는 엔티티링킹 모델은 Two stage로 이루어진다. 첫째, BERT로 이루어진 bi-encoer로 mention의 컨텍스트와 위키피디아의 모든 엔티티 description간의 스코어를 구해 ranking을 매긴다. 둘째, cross-encoder로 mention context와 entity description을 한번에 담아 스코어를 구해 그중에 가장 높은 스코어를 가지는 엔티티가 gold entity(정답)가 된다.

논문에서 가장 말하고 싶은 것은 scalable하다는 것인 것 같은데, 학습을 할때는, bi-encoder를 모두 돌려야 되지만, 학습 후, 엔티티 encoder로 미리 모든 엔티티를 인코딩 해두고 dense space를 만들어두고 mention context만 인코딩하여 이미 인코딩 된 모든 위키피디아 엔티티에 대해 nearest neighbor search가 가능하단 점이다. (실제로 코드를 보면 서치 툴로 페이스북리서치 본인들이 만든 faiss를 사용하는 것을 볼 수 있다. 논문에도 나오고.. 써봤는데 빠르긴 하다.)  그렇게 ranking된 Top K 엔티티에 대해서만 좀더 세밀하게 cross-encoder를 적용하여 링킹한다.

따라서 좀 더 정확하고, 더 효율적으로 큰 스케일의 엔티티 링킹을 할 수 있게 되었다는 것이다.

엔티티의 description을 이용하기 때문에 물론 제로샷 링킹도 가능하겠다.
(추가로 biencoder 성능향상을 위해 학습할 때 crossencoder로부터 지식증류를 이용했다고 한다.)


<br>


# Content

- Bi-encoder의 구성

![png](/images/1.PNG "그림1"){: width="100%" height="100%"}  

스코어는 각 인코더에서 뽑은 representation을 내적해서 구한다.

![png](/images/2.PNG "그림1"){: width="100%" height="100%"}  

따라서 로스는 다음과 같이 구성된다.(Softmax)

그리고 학습에 hard negative mining을 사용하였다고하는데, 엔티티링킹은 true positive만 학습데이터로 주어지기 때문에 true negative에 대한 학습도 필요하다. 이를 위해 mining을 하는 것이다.

- Cross-encoder의 구성

Cross encoder는 mention context와 candidate entity description과 결합하여 입력으로 들어간다.

![png](/images/3.PNG "그림1"){: width="100%" height="100%"}  

스코어는 인코더에서 뽑아낸 representation에 linear layer(W)를 적용해 뽑아낸다.

로스는 biencoder와 동일하게 구성된다고 보면된다.

Qualitative 분석에서 biencoder는 ronaldo를 브라질 축구선수로 링킹(Top score)했지만, cross-encoder를 이용해 정답을 링킹했다. 이로써, context, entity description 간의 관계, attention을 이용해 인코딩 하는 것이 정확하게 링킹하는 데에는 중요하다고 볼 수 있다. 

이어지는 논문으로 페이스북이 ELQ를 발표했는데, biencoder만을 이용하여 성능보단 속도, 효율에 집중하여 question에 대한 엔티티링킹을 구현했다. 솔직히 본 논문이랑 별 차이가 없긴 하지만, biencoder만으로도 충분한 성능이 나고 qa에 적용하려면 속도가 중요하므로 굳이 cross encoder를 사용하지 않고, biencoder를 썼다는 정도 인 것 같다.
