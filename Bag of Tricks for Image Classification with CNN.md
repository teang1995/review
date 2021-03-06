# Bag of Tricks for Image CLassification with CNN review

## Abstract
- 최근 image classification의 개선은 data augmentation이나 optimization method덕분이라고 여겨짐.
 -  하지만, 대부분의 개선은 trick들에 의한 것.
 -  해당 paper에서는 그것들을 종합하고, 그것들의 성능을 평가함.
 -  이런 trick들로 ResNet-50을 ImageNet으로 학습시켰을 때, top-1 valid accuracy가 75.3%에서 79.2%까지 증가함.
 -  classification accuracy의 개선으로 detection이나 segmentation의 전이학습 또한 개선됐음.

## Introduction
-  해당 paper에서는 학습 과정에서 자주 사용되지만 잘 정리되지 않았던 trick들에 대해 실험함.
-  이러한 trick (stride를 바꾸거나, learning rate schedule을 조정하거나)들은 사소하지만 모이면 큰 차이를 만듦.

### Paper Outline
1.  section 2에서 학습의 baseline을 구성함.
2.  section 3에서 새로운 hardware 에서 유용한 trick들을 소개함.
3.  section 4에서 ResNet을 살짝 변경한 모델의 구조를 review하고, 새로운 것을 제시함.
4.  section 5에서 추가적인 학습 과정의 개선에 대해 논의함.
5.  마지막으로, section 6에서 이러한 model의 개선이 전이학습에 도움이 되는지 알아봄.

## Training Procedures   
![알고리즘1](https://i.ibb.co/GJ2cmCx/algo1.png)
- 위의 알고리즘에 의해 학습을 진행함.
- 매 iteration에서 b(batch_size) 개의 이미지를 뽑아 계산하고, network parameter에 반영.
-  K epoch 학습 진행 후 학습을 멈춤.

### Baseline Training Procedure
1.  이미지를 임의로 샘플링해 32bit float형으로 디코딩함.
2.  이미지를 random - crop하여 224 x 224 size로 resize함.
3.  0.5의 확률로 horizontal flip 수행.
4.  hue,saturation,brightness를 [0.6,1.4] 에 따른 정규화함.
5.  pca noise를 추가해 normalization해줌.
6.  RGB channel을 각각 [123.68,58.393], [116.779,57.12],[103.939,57.375]에 따라 정규화함.

- Validation 중 짧은 쪽을 256 pixel로 resize하고, 224 x 224 size로 center crop함. 
- conv layer, fcn의 weight들은 Xavier Initialization을 따름. 
-  batch normalization의 감마는1, 베타는 0으로 초기화.
-  모든 bias는 0으로 초기화됨.
-  optimizer는 NAG(Nesterov Accelerated Gradient)가 사용됨. 
-  8개의 Nvidia V100 Gpu가 사용됐고, 배치 사이즈는 256으로 설정
-  learning rate는 0.1로 초기화되고, 30 epoch마다 1/10으로 줄어듦.
- ImageNet(ISLVRC2012-1000classes)로 학습 진행.
### Experiment Results
![table2](https://ifh.cc/g/Fy8gB.png)

## Efficient Training
- GPU의 발전으로 더 큰 batch size ,  더 낮은 numerical precision을 사용하는 것이 효율적이게 됨.
-  해당 섹션에서는 위의 두 방법들을 model의 정확도를 희생시키지 않고 사용하게 할 수 있는 다양한 기술에 대해 논의하고자 함.

###  A. Large-batch training
- convex문제를 풀 때, batch size가 증가하면 수렴이 느려진다고 알려져있음.
- nn에서도 비슷한 결과가 나타남이 보여짐.
- 이러한 문제를 해결하기 위해 제안된 방법들을 소개

	#### A1. Linear scaling learning rate
	 - SGD 에서 batch size를 키우면 그 기댓값은 일정하지만, variance는 줄어듦.
	 - 배치 크기를 키우면, 노이즈가 줄어든다는 의미.
	 - 이를 통해 learning rate를 키울 수 있게 됨.
	 - 예를 들면 기존의 batch size가 256 , learning rate가 0.1일 때, 증가된 batch size를 b라고 하면 learning rate를 0.1 * b/256으로 사용한다는 의미.

	#### A2. Learning rate warmup
	- 학습 초반엔 모든 파라미터들이 랜덤으로 초기화돼있음.
	- 너무 큰 learning rate를 사용하는 것은 수치적으로 불안정한 상태를 야기함.
	- 따라서, 매우 작은 learning rate로 출발해  특정 시점까지 원하는 learning ratedp 도달하게 함.
	- Goyal은 0에서 시작해 **선형적**으로 learning rate를 증가시키는 방법을 제안.(5epoch까지 warm-up을 적용함.)

	#### A3. zero $\gamma$
	- resNet은 여러 개의 residual block으로 구성됨.
	- 입력이 $x$ 일 때, 출력은 $x + block(x)$으로 표현할 수 있음.
	- block의 마지막엔 BN(batch normalization) 적용.
	- BN은 입력값 $x$에 정규화를 적용한 결과를 $\hat x$라 하고,  이에 대해 $\gamma\hat x + \beta$ 반환.
	- $\gamma$,$\beta$는 각각 1,0으로 초기화되는 학습 가능한 파라미터들.
	- 따라서 초기에는 입력이 그대로 전달돼 학습이 빨라지게 된다.

	#### A4. No bias decay
	- weight decay는 보통 weight와 bias를 가지는 모든 학습되는 파라미터에 적용됨.
	- 이는 모든 파라미터에 L2 regularization을 적용하는 것과 비슷하여, 그 값들을 0에 수렴하게 함.
	- 이는 conv layer, fc layer에만 적용되고,BN에는 적용되지 않음.

### B. Low- precision training
- 신경망은 대부분 32bit floating point type으로 학습 진행.
- 이는 저장과 연산에도 해당되는 내용.
- 최근의 하드웨어는 더 작은 크기의 데이터 타입의 연산에서 좋은 결과를 보임.
- 예를 들면, V100을 기준으로 FP32에서 FP16으로 자료형을 바꾸었을 때 학습 속도가 2~3배 가속됨을 확인할 수 있었음.
- 이러한 장점에도 불구하고, FP16은 표현할 수 있는 범위가 적어 학습 과정에서 문제가 발생함.
- Micikevicius는 FP16으로만 구성된 신경망을 구성하고, 학습 과정에서 FP32로 복사하는 모델을 제안함.

### Experiment Results
![Table3](https://ifh.cc/g/A0eCz.png)
batch size를 증가시키고, FP16을 사용했을 때의 성능 향상.![enter image description here](https://ifh.cc/g/gJRxr.png)
batch size를 증가시키고, 제안된 heuristics를 적용시켰을 때의 성능 향상.


## Model Tweaks
- tweaks는 conv layer를 약간 수정하는(ex. stride) 작업을 말함.
- 이는 기존 모델의 계산 복잡도를 바꾸지만, 모델의 정확도 향상에 유의미한 영향을 주기도 함.
-  해당 섹션에서는 ResNet에 대한 Tweaks에 대해 이야기함.

### ResNet Architecture

![enter image description here](https://ifh.cc/g/5hXoz.jpg)

위의 base line을 tweak한 ResNet-B와 ResNet-C를 본 후, 다음 섹션에서는 ResNet-D를 제안함.

![enter image description here](https://ifh.cc/g/g2aax.jpg)

### ResNet-B
- stride = 2인 layer를 1x1conv layer와 순서를 바꾸어 손실되는 정보가 없게 함.

### ResNet-C
- Input Stem을 바꾼 tweak.
-  Input Stem의 7x7 conv를 여러 개의  3x3 conv로 변경.

### ResNet-D
- ResNet-B에 영감을 받았음.
- ResNet-B의 1x1,stride = 2인 conv layer에서 3/4의 정보가 손실됨.
- 이를 avg-pooling으로 대체하여 손실되는 정보가 없도록 함.

![enter image description here](https://ifh.cc/g/9Sngm.png)
ResNet baseline과 ResNet - B,C,D의 성능 비교.

## Training Refinements
해당 섹션에서는 모델의 정확도를 위한 training refinements들에 대해 서술함.

### Cosine Leaning Rate Decay
- Learning rate의 조절은 학습에 매우 중요함.
- 앞서 언급된 warm - up 이후, 초기값으로부터 learning rate를 감소시킴.
- 가장 많이 사용되는 방법은0.1의 초기값에서  30 epoch마다 일정 비율로 감소시키는 "step decay".
- 2 epoch마다 0.94의 비율로 감소시키기도 함.
- Loshchilov은 cosine annealing기법을 제안.
	1.초기값이 0인 learning rate에서 cosine함수값에 따라 값을 결정함.
    2. 전체 batch수를 T, 현재 batch 수를 t, learning rate의 초기값을 $\eta$라 함.
    3. 그 때, batch t에서의 learning rate는 다음과 같음.
    $\eta_t=\frac 1 2(1+cos(\frac \pi T t))\eta$ 
- step decay와 다르게 learning rate가 천천히 감소하는 것을 확인할 수 있음.
- 이는 또한 학습 과정을 개선시킴.
![enter image description here](https://ifh.cc/g/k1M05.png)
cosine decay와 step decay의 성능 비교

### Label Smoothing
-  image classification을 위한 신경망의 마지막 fcn은 class의 갯수와 동일하게 설정, 이를 K라 함.
- -cross entropy에 따라서 loss를 구해보면, $l(p,q)=-\sum_{i=1}^{K} {p_ilogq_i}$이다. 
- cross entropy의 정의에 따라, $l(p,q) = -log(q_y) = -z_y + log( \sum_{i=1}^{K}exp(z_i))$임을 알 수 있다. 
- 최적화된 해는 $z_y = inf$ 일 때(loss가 최소일 때)임을 알 수 있는데, 이 경우 $z_y$는 무한정 커지고 나머지 확률들은 무척 작아져 overfitting 을 야기하게 된다.
- 이러한 이유로, label smoothing이 제안되었다. 

### Knowledge Distillation
- 먼저 잘 학습된 model(teacher model)를 새로 학습시킬 model이 학습하도록 하는 것.
- 학습 과정에 distillation loss를 추가해 teacher model과의 차이를 줄이도록 한다. 
- p가 정답의 확률  분포, z와 r은 각각 student,teacher model의 fc layer의 출력이라 하자.
- 이  때 새로운 loss가 다음과 같이 정의된다.
- $L = l(p,softmax(z)) + T^2l(softmax(r/T),softmax(z/T))$
- T는 teacher model의 prediction을 빼내기 위해 softmax의 출력을 smooth하기 위한 hyper-parameter이다.

### Mixup Training
- Mixup이란, 이미지 두 개를 섞는 augmentation기법.
- (img1,label1),(img2,label2) 두 개의 sample이 있을 때, 다음과 같은 하나의 샘플을 만들어 학습에 사용하는 것을 말함.
img3 = $\lambda$img1 + $(1-\lambda)$img2
label3 = $\lambda$ label1 + $(1-\lambda)$label2
- $\lambda$는 [0,1]에서 Beta($\alpha$,$\alpha$)의 분포를 따라서 임의로 추출된 변수.

### Experiments Results
![enter image description here](https://ifh.cc/g/WCtkn.jpg)

- 네 개의 refinements들에 대해 실험 진행.
	1. cosine learning rate decay 적용함.
	2. $\epsilon = 0.1$로 두고 label smoothing.
	3. distillation 위해 T=20으로 두고, teacher model은 ResNet-D-152가 사용됨.
	4. mixup model에서 $\alpha = 0.2$로 두고 200epoch동안 학습 진행.

- distillation은 ResNet에선 잘 작동했지만 Inception-V3나 MobileNet에선 잘 적용되지 않았음.
-  다른 데이터셋(MIT Places365 dataset)에도 적용됨을 확인할 수 있었음. (Table7)



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI4MTY1MzYzLDE0MDQ1NTQ2MTMsLTIwMD
MwMzk3NiwyMTQwODc0MDY4LC0yMDU1NDg4NjgyLDc5MDk4MDc4
NiwtNzI5ODg0MzkwLDE1ODY3NDc5MzgsLTE5MjU3MTE3MzYsLT
QzMTQyNTE0MywtNjY3NjQ3NDk2LC0yNjEyMjc4OTEsMTI1MTg2
MzU0NywxNDYwMTc0MTE3XX0=
-->