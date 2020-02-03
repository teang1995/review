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
2.  이미지를 random - crop하여 224 * 224 size로 resize함.
3.  0.5의 확률로 horizontal flip 수행.
4.  hue,saturation,brightness를 [0.6,1.4] 에 따른 정규화함.
5.  pca noise를 추가해 ???
6.  RGB channel을 각각 [123.68,58.393], [116.779,57.12],[103.939,57.375]에 따라 정규화함.

- Validation 중 짧은 쪽을 256 pixel로 resize하고, 224 * 224 size로 center crop함. 
- conv layer, fcn의 
-  batch normalization의 감마는1, 베타는 0으로 초기화.
-  모든 bias는 0으로 초기화됨.
-  optimizer는 NAG(Nesterov Accelerated Gradient)가 사용됨. 
-  8개의 Nvidia V100 Gpu가 사용됐고, 배치 사이즈는 256으로 설정
-  learning rate는 0.1로 초기화되고, 30 epoch마다 1/10으로 줄어듦.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUyMTY1OTU0MywtNjY3NjQ3NDk2LC0yNj
EyMjc4OTEsMTI1MTg2MzU0NywxNDYwMTc0MTE3XX0=
-->