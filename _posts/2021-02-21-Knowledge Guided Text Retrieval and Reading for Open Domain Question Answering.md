---
title:  "[질의 응답] Knowledge Guided Text Retrieval and Reading for Open Domain Question Answering"
excerpt: "Knowledge Guided Text Retrieval and Reading for Open Domain Question Answering  논문 소개"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/logo.jpg

categories:
  - Question Answering
tags:
  - Question Answering
  - Open domain Question Answering
  - Knowledge graph
  - Knowledge base
  - Wikipeda, Wikidata
last_modified_at: 2021-02-21T08:06:00-05:00
---
# Introduction

본 논문은 open domain qa 를 위해, Wikipedia와 Wikidata(지식베이스)를 이용해 Question에서 뽑아낸 정보(TF-IDF, entity linking)를 이용해 뽑아낸 Wikipedia의 passage들로 knowledge graph를 만든다. 그렇게 검색을 통해 얻은 graph로 GCN(Graph Convolution Neural net)을 이용해 graph의 관계를 반영한 representation을 뽑아내서 passage들 중 answer span이 있을 것으로 예상되는 passage의 span을 찾아내는  structure를 이용하여 실험결과를 제시하고 좋은 성능을 나타낸다고 설명하고 있다.

가장 핵심은 wikidata(KB)와 wikipedia에서 뽑아낸 passage graph를 GCN을 통해 서로 연결된 passage 간의 관계들로부터 종합적으로(isolation없이) representation을 뽑은 것이 Answer가 있는 passage를 찾고 span을 찾는데 큰 도움이 되었다는 것이다.


<br>


# Content

![png](/images/q1i1.PNG "그림1"){: width="100%" height="100%"}  

위의 그림을 보면 Question에 대해 GraphRetriever를 통해(왼쪽) passage그래프를 구성하고 GraphrReader(오른쪽) 를 통해 representation을 뽑아내는 모습이다.

GraphRetriever에 대해 설명하자면, 다음과 같다.

1. 우선 Seed가 되는 Passage들을 찾는다.

엔티티링킹을 해서 링킹된 엔티티에 해당되는 Wikipedia article들 + Question의 TF-IDF를 이용한 검색을 이용해 얻어낸 article들(상위 rank의 article K개)의 첫번째 passage를 Seed Passage들로 한다.

2 .Graph를 확장한다.

첫째, Seed Passage와 relation을 가지는 article들을 Wikidata(KB)를 통해 찾고, 그것들의 첫번째 passage를 graph에 포함시킨다.

둘째, Seed Article들의 첫번째 passage들을 제외한 나머지 passage들을 BM25를 이용해 랭크를 매겨 상위에 rank되는 S개를 포함한다.

3. 2번 항목 반복.(iteration)

1번 항목에서 뽑은 Seed Passage들이 P(0)이라 한다면 2번에서 뽑은 Passage들이 P(1)이라 할 수 있다. 이렇게 나온 P(i)로 2번 항목을 iteration한다. 총 n개의 passage가 나올때까지.

GraphReader에 대해 설명하자면, 다음과 같다.

1. Bert(passage Encoder)를 통해 passage를 인코딩, 별개의 embedding matrix를 만들어 relation을 인코딩.

2. 인코딩하여 뽑은 representation을 GCN구조를 이용해 graph관계를 이해한 passage별 representation을 다시 뽑는다. 수식은 다음과 같다.  f 는 concatenation으로 보면된다. 저자가 다른 거창한 연산(element wise 곱)보다 잘 작동한다고 한다.

![png](/images/q1i2.PNG "그림1"){: width="100%" height="100%"}  

![png](/images/q1i3.PNG "그림1"){: width="100%" height="100%"}  

Z는 뽑아낸 단락별 representation이라 보면 되고 어떤 단락이 정답일지 probability를 뜻한다고 보면된다..

![png](/images/q1i4.PNG "그림1"){: width="100%" height="100%"}  

Softmax를 이용해 뽑은 단락중에(maxlen정해짐) 어떤 토큰이 start, end토큰일지 probability 나타낸다고 보면된다.

그래서 Objective function은 다음과같다.

![png](/images/q1i5.PNG "그림1"){: width="100%" height="100%"}  

passage 선택하는 것이랑 span찾는 부분을 더해서 maximum likelihood object function를 구성한 것을 알 수 있다.


