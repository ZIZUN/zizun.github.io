---
title:  "[pytorch] eval시 gpu memory 부족 해결"
excerpt: "eval시 gpu memory 부족 해결"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/logo.jpg

categories:
  - pytorch
tags:
  - pytorch

last_modified_at: 2021-08-23T09:06:00-05:00
---

간단하게,

with torch.no_grad() 로 감싸고 eval을 진행하면 된다.

pytorch의 autograd를 끔으로써 gradient계산에 필요한 메모리를 날려주게된다.

model.eval()은 dropout이나 batchnorm같은 것을 추론시에 빼고 진행하기 위해 사용한다.

+ 추가로 torch.cuda.empty_cache() 를 이용해 캐쉬된 학습데이터의 메모리를 제거하는 방법도 있다.