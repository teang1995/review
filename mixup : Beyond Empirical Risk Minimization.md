# mixup : Beyond Empirical Risk Minimization

## Abstract

- neural network은 강력하지만, 대립 관계의(옳지 않은) 예에 민감하게 반응하거나, 기억하는 등의 예상치 못한 결과를 보일 때도 있었다.
- 본 paper에서는, 위의 문제를 다루기 위해 *mix-up* 기법을 제안한다.
- 이는 neural network를 data와 label의 convex combination으로 학습시키는 방법이다. 
- ImageNet-2012, CIFAR-10 등의 dataset에 대해 실험을 진행했고, *state-of-the-art*기법의 모델에 대해서 성능을 개선할 수 있다는 것을 확인했다.
- 또한 학습에 방해되는 label을 기억하는 문제를 줄이고, adversarial example에 대한 강인함이 증가하며, GAN의 학습을 안정화시킴을 확인했다.

## Introduction

- Large deep neural network는 computer vision, speech recognition, reinforce learning 등의 분야에서 많은 발전을 가능하게 했다.
- 이러한 neural network들은 두 가지 공통점을 가진다
		 -training data에 대한 average error를 최소화시키는 방향으로 학습된다.
		 -training examples의 수가 늘어남에 따라 neural network의 크기도 선형적으로 증가한다.
- 기계학습의 고전적인 이론은 ERM의 수렴은 training data의 수에 따라 neural network의 크기가 증가하지 않으면 보장된다고 말했다.
- ERM은 큰 neural network가 training data를 기억할 수 있게 했다.
- 하지만, training data의 분포에서 벗어난 data(adversarial example)이 평가받을 땐, 정확한 예측을 하기 어려워진다.
- 따라서, ERM을 이용한 학습은 training data의 분포에서 조금이라도 벗어난 data는 설명하기 어려워진다.
- 이러한 방법을 해결하고자, VRM에 의해 공식화된 data augmentation이라는 방법이 소개되었다.
- VRM은 인간의 지식은 training data의 근처까지 생각하도록 요구된다고 했다.
- 추가적인 실제의 예들은 training data의 근처로부터 확장될 수 있다고 했다.
- data augmentation이 눈에띄는 발전을 이룬 것은 맞지만, 이 과정은 dataset에 의존적이고, 전문가적 지식을 요구하였다.
- 또한, data augmentation은 근처의(유사한) data들이 같은 클래스로 분류될 것이라고 가정했고, 유사해 보이는 data가 다른 class에 해당하는 경우를 상정하지 않았다.


## From Empirical Risk Minimization To Mixup

- 우리는 지도학습에서 결합확률분포 $P(X,Y)$를 따르는 feature vector $X$ 와 target vector $Y$의 관계를 설명해주는 함수 f를 찾는 데에 관심이 있다.
- 이것 때문에, $f$ 의 예측 $f(x)$와 실제 label $y$의 차이에 페널티를 부과하는 loss function $l$을 정의했다.
- 그리고 우리는 expected risk라고도 알려진 아래의 식과 같은  data분포 $P$에 대해 
loss function $l$의 평균을 최소화시켰다. 
$$R(f) =\int l(f(x),y) \, dP(x,y) $$
 
- 불행하게도, data분포 $P$는 대부분의 실용적인 상황에서 알 수 없었다.
- 대신, 우리는  보통은 $D =${(x_i)}$

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM4NTI1OTU3MCwtMTgwNjc0OTI1XX0=
-->