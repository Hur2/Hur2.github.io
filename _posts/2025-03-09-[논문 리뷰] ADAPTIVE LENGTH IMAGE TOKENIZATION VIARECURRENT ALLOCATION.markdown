---
layout: post
title:  "[논문 리뷰] ADAPTIVE LENGTH IMAGE TOKENIZATION VIA RECURRENT ALLOCATION"
date:   2025-03-09 23:18:35 +0900
categories: jekyll update
---

## ADAPTIVE LENGTH IMAGE TOKENIZATION VIA RECURRENT ALLOCATION


>  1. Background
> 

Representation learning이란 입력 데이터(이미지, 오디오 등)에서 의미있고 유용한 정보를 추출하도록 학습하는 분야이다. 이 유용한 정보는 압축된 벡터의 형태로 표현되고, 이는 ‘latent representation’, ‘latent token’ 등의 여러 표현으로 불린다.

이 논문은 이미지를 압축하는 것에 관심을 가지고 있다. image embeddings에는 다음과 같은 종류가 있다.

- encoder-only methods
    - contrastive learning
    - self-distillation
- encoder-decoder methods
    - reconstruction

이 논문은 reconstruction 방식에 대해서 다룬다.

![스크린샷 2025-03-14 오후 11.45.54.png](https://hur2.github.io/_posts/2025-03-09-files/image-10.png)

reconstruction은 ‘input 이미지’와 encoder/decoder를 거친 ‘output 이미지’가 최대한 유사하도록 학습하는 방식이다. 이 과정에서 encoder는 input 이미지를 latent token으로 잘 압축하도록 학습되고, decoder는 latent vector를 활용하여 이미지를 잘 생성하도록 학습된다.

>  2. Idea
> 

![스크린샷 2025-03-14 오후 11.49.53.png](https://hur2.github.io/_posts/2025-03-09-files/image-11.png)

저자들이 관심을 갖은 부분은 이 latent token부분이다. 이 부분의 길이가 고정되어 있다는 점이 문제라고 지적하였다.

이유는 다음과 같다.

1. Task 마다 요구하는 latent representation compression 방식이 다르다.
: classification과 reconstruction 는 다른 방식의 compression을 필요로 한다. classification은 추상적인 표현(shape/texture)만으로도 가능한 반면, reconstruction는 손가락 같은 더 디테일한 데이터를 필요로 한다. 즉, reconstruction은 더 데이터가 밀집된 토큰 방식을 필요로 한다.
2. 이미지 마다 다른 양의 토큰을 필요로 한다.
: 이미지의 entropy, context and familiarity에 따라 다른 양의 토큰이 필요하다. 쉽게 말해, 복잡한 이미지는 더 많은 토큰을, 단순한 이미지에는 더 적은 토큰을 필요로 한다는 것이다. 이렇게 적응적인/동적인 표현이 인간 지능과 더 유사한 방식이라고 설명한다.

그러므로, latent token의 길이를 조절하고 싶으나, 2d inductive bias라는 한 가지 문제가 더 있다. 

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-1.png)

VIT를 예시로 들면, 네트워크 전 과정동안 이미지의 patch의 수와 token의 수가 1대 1로 대응된다. patch 수에 의해서 token 수가 제약을 받기 때문에, 우리가 원하는 adaptive한 길이의 latent token이 되는 것을 막는다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-2.png)

이에 저자들은 PERCEIVER ([Jaegle et al., 2021b;a](https://arxiv.org/abs/2103.03206)) 라는 paper를 소개한다. 해당 페이퍼는 어떤 크기/종류(음성/이미지 등)의 입력이 오더라도, 1-d 고정된 길이의 latent array에 정보 주입 가능하도록 제안하였다. 특정 데이터에 대한 inductive bias를 없애고, muti-modality를 위한 방식이다. 이러한 방식을 차용해 기존 2d token을 1-d으로 distillation 시키는 아이디어를 얻는다.

정리를 하면, 

문제1: 2d inductive bias가 존재한다. ⇒ 2d token을 1-d로 distillation 시킨다.

문제2: 어떻게 길이를 조절할 것인가? ⇒ distillation을 recurrent하게 수행함으로써, 길이를 늘린다.

![스크린샷 2025-03-15 오후 1.27.36.png](https://hur2.github.io/_posts/2025-03-09-files/image-12.png)

전체적인 구조는 위의 그림과 같다.

>  3. Method
> 

디테일한 과정을 살펴보자. 수식으로는 아래 그림과 같이 표현된다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-3.png)

1th iter 을 보자.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-4.png)

VQGAN encoder를 통해 나온 2d token(16x16개)과 초기화된 길이가 32인 1d token을 latent-distillation encoder의 입력으로 넣는다. latent-distillation encoder는 단순한 8개 layer의 self-attention으로 이루어져 있다. 즉, encoder를 거치게 되면 2d token에 대한 정보가 1d token에 distillation이 된다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-5.png)

이제는 역으로 길이가 32인 latent 1D token을 과 masked 2D tokens을 latent-distillation decoder에 넣는다. latent-distillation decoder은 latent 1D token을 가지고 2D token을 잘 복원할 수 있도록 학습되어진다. 즉, VQGAN encoder를 통해 나온 2d token과 latent-distillation decoder의 output으로 나온 2d token이 유사하도록 학습되어진다.

이렇게 2th iter로 넘어간다. 1th 와는 약간 다른 점이 있다.

![스크린샷 2025-03-15 오후 1.38.39.png](https://hur2.github.io/_posts/2025-03-09-files/image-13.png)

2D token에는 마스킹이 씌워진다. 기존 1th decoder에서 reconstruction이 잘 된 부분을 masking을 씌운다. 이는 이미 distillation이 잘 된 부분임으로, 아직 distillation하지 못한 정보를 주입시키기 위함이다. 그리고 1th encoder에서 생성된 latent 1D token 에 1d token을 concat하여  latent-distillation encoder에 입력으로 넣는다. 

한번의 iteration이 진행될 때마다, latent 1D token의 길이는 32씩 증가한다. iteration은 최대 8번까지 진행함으로, 길이는 32~256으로 구성된다. iteration을 중단시키는 조건은 여러가지 방식을 소개하는데, 자동화된 방법으로는 latent-distillation decoder의 output인 2d token의 reconstruction loss가 thresh_hold보다 작아지면 멈추도록 설정한다. 이를 token selection criteria라고 부른다.

>  4. Experiments
> 

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-6.png)

복잡한 이미지일수록, reconstuction을 위해서 더 많은 token이 필요하다.

![스크린샷 2025-03-15 오후 2.01.05.png](https://hur2.github.io/_posts/2025-03-09-files/image-14.png)

학습에 사용된 분포를 가진 이미지는 적은 데이터도로 표현이 가능하나, 분포와 먼 데이터들을 표현하기 위해서는 더 많은 토큰이 필요하다. 이러한 사실들은 이미지마다 다른 토큰을 요구함을 보여주고 있다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-7.png)

더 적은 모델, 더 적은 데이터(학습 자원의 한계로) 학습했음에도 불구하고 다른 모델들과 큰 차이가 나지는 않는 성능을 보여준다.

 ImageNet-1K Classification에 대한 Linear Probing 수행

- 두 가지 관찰 포인트

(a) Recurrent Processing의 역할

encoder 1 time:  32 latent tokens achieves 55.6%

encoder 2–3 times improves this to 59.5%.    

(b) classfication에는 64 ~ 128개의 1d token이 충분하다

256 개 토큰보다 64 to 128 토큰에서 더 좋은 성능을 내는 것을 확인했다. classification을 위해서는 초기 토큰으로도 충분하고, 추가적인 토큰들은 reconstruction에서 중요한 역할을 하는 것으로 분석했다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-8.png)

기존에 2d inductive bias가 있는 모델들은 patch들에 제약을 받아서, attention map이 match에 영향을 받았으나, 저자들의 모델에서는 segmentation의 역할을 수행 가능함을 보여준다.

![image.png](https://hur2.github.io/_posts/2025-03-09-files/image-9.png)

recurrent를 반복 할수록, 각 토큰들이 더 세세한 부분에 집중하는 것을 알 수 있다. 이는 recurrent의 중요한 역할을 확인할 수 있는 자료이다. recurrent를 반복할 때마다, 추가로 토큰을 제공함으로써 기존의 토큰들이 더 자신의 분야에 집중할 수 있도록 한다는 것이 저자들의 설명이다. 각 토큰들이 바닥/논 등 특정 부위에 타겟된 것을 알 수 있다.