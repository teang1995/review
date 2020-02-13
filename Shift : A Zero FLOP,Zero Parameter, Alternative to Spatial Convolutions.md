# Shift : A zero FLOP, Zero Parameter Alternative to Spatial Convolutions

## Abstract
- CNN은 공간 정보를 종합하기 위해 convolution연산에 의존한다.
- 하지만, spatial convolution은 model의 size와 연산에 비용이 많이 든다.
- 본 paper에서는 spatial convolution을 대체할 수 있는 parameter,FLOP이 없는 "shift"연산을 제안한다.

## Introduction and Related Work
- CNN은 classification, detection, style transfer와 같은 모든 computer vision영역에 적용 가능하다.
- 이러한 이유로 CNN은 IOT에 적용되기도 한다.
- 하지만 이는 CNN의 메모리에 제한을 준다.
- 이러한 이유로, model의 정확도를 유지하면서 CNN의 크기를 줄이는 데에 집중했다.
- CNN에서 사용되는 spatial convolution은 계산과 model의 크기에 모두 큰 비용이 든다.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAyMzc0ODY3NV19
-->