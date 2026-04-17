# EfficientNet-B0 구현

## 1. EfficientNet-B0를 구현하게 된 동기  
다양한 학습 전략을 실험하고 싶었지만 모델 학습에 소요되는 시간이 상당해 반복적인 실험에는 무리가 있었습니다. 비교적 가벼우면서도 성능이 좋은 모델이 있었으면 했고, 탐색 과정에서 ResNet-50 대비 적은 파라미터와 경쟁력 있는 성능을 가진 EfficientNet-B0를 발견할 수 있었습니다.

해당 모델에 대해선 이전에 들어본 경험이 있었으며, 효율적인 모델 스케일링 방식을 제안한다는 점에 흥미를 느껴 이번에 구현하기로 결정했습니다.

## 2. 구현에 참고한 논문
**[1]   EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks**
([논문 링크](https://arxiv.org/abs/1905.11946))

이름 그대로 EfficientNet을 소개한 논문입니다. Neural Architecture Search(NAS)로 최적의 모델 구조를 발굴하고, depth와 width, resolution을 체계적이고 종합적으로 스케일링해 모델 성능을 향상시킨 점이 인상 깊었습니다.

또한 EfficientNet-B0부터 B7까지 다양한 모델을 제시하며 타 모델들과 이미지 분류 정확도, 추론 시간, 파라미터 수, FLOP 등을 비교하는데, 테이블과 그래프 시각화가 정말 잘 되어 있어 이해하기 편했습니다. 멋진 논문이었습니다.

**[2] MnasNet: Platform-Aware Neural Architecture Search for Mobile** ([논문 링크](https://arxiv.org/abs/1807.11626))

EfficientNet 모델 패밀리의 백본이 되는 EfficientNet-B0는 어떻게 설계된 걸까요? 바로 이 논문에서 제안한 방법론을 활용했다고 합니다. EfficientNet 논문에서도 해당 방법론을 간략히 다루고 있기 때문에 보다 자세한 내용이 궁금하시면 이 논문을 보면 될 것 같습니다.  
EfficientNet의 MBConv Block이 궁금하신 분들도 이 논문을 참고하시면 될 것 같습니다.

**[3] MobileNetV2: Inverted Residuals and Linear Bottlenecks** ([논문 링크](https://arxiv.org/abs/1801.04381))

Inverted Bottleneck Residual block의 구조와 개념을 이해할 때 참고한 논문입니다. 2번 논문과 함께 구현에 많은 도움이 되었습니다.  
초반에 직관적으로 이해하기 어려운 내용이 일부 있었습니다. 읽으면서 아직 공부가 더 필요한 것 같다고 느꼈습니다.

**[4] Squeeze-and-Excitation Networks** ([논문 링크](https://arxiv.org/abs/1709.01507))

SE Block 구현을 위해 참고한 논문입니다. 메커니즘이 약간 Attention과 유사해서 흥미로웠습니다. Attention에서 각 입력 간 관계를 학습하고 가중치를 주는 것처럼, SE Block에서도 이와 유사하게 입력 피처 맵의 통계량 간 관계를 학습하고 이를 기반으로 가중치를 계산하는 과정이 있었습니다. 읽으면서 '만약 CNN 전용 Attention이 있다면 이런 느낌이지 않을까?' 하는 상상을 했습니다. 재밌는 논문이었습니다.

## 3. 모델 구현
Pytorch에 구현된 모델과 동일한 구조, 동일한 파라미터 수를 가지는 EfficientNet-B0을 구현했습니다.  
[구현3] EfficientNet-B0.ipynb 파일 참고 바랍니다.  

## ~~4. 학습 결과 (25.09)~~
~~여러 차례 실험 후 가장 높게 나온 성능을 기록했습니다.~~

## 5. 재학습 결과 (26.01)
**⚠️주의**: 학습 데이터셋은 **Imagenette**으로, ImageNet의 축소 버전입니다.  

[적용한 데이터 증강]
- RandomHorizontalFlip(p=0.5)
- ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.05)

| No. | 실험 모델 | 파라미터 | 증강 | Val Acc | 학습 상세 | 학습 시간 (L4 GPU) |
|:---:|:--------:|:---------:|:----:|:-------:|:---------:|:------------------:|
| 1 | VGG16         | 138M | X | 78.98%    |LR=1e-4, Batch_size=64, Epochs=60 | 92분 (1.5분/Epoch) |
| 2 | VGG16         | 138M | O | 82.37%    |LR=1e-4, Batch_size=64, Epochs=60 | 92분 (1.5분/Epoch)|
| 3 | ResNet-50 | 25M | X | 86.93% |LR=1e-4, Batch_size=64, Epochs=60 | 57분 (1분/Epoch)|
| 4 | ResNet-50 | 25M | O | 87.34% |LR=1e-4, Batch_size=64, Epochs=60 | 57분 (1분/Epoch)|
| 5 | **EfficientNet-B0** | 5.3M | X | **85.61%** |LR=3e-4, Batch_size=64, Epochs=60 | 36분 (0.6분/Epoch)|
| 6 | **EfficientNet-B0** | 5.3M | O | **87.80%** |LR=3e-4, Batch_size=64, Epochs=60 | 36분 (0.6분/Epoch)|
| 7 | VGG16-bn      | 138M | X | 89.71%    |LR=1e-4, Batch_size=64, Epochs=60 | 112분 (1.9분/Epoch)|
| 8 | VGG16-bn      | 138M | O | 90.75%    |LR=1e-4, Batch_size=64, Epochs=60 | 112분 (1.9분/Epoch)|

<img width="790" height="590" alt="image" src="https://github.com/user-attachments/assets/d3e70b6f-71ee-4704-aeb9-92bd9a0d725a" />

증강 여부를 기준으로 정렬하면 다음과 같습니다.

| No. | 실험 모델 | 파라미터 | 증강 | Val Acc | 학습 상세 | 학습 시간 (L4 GPU) |
|:---:|:---------:|:--------:|:----:|:-------:|:---------:|:------------------:|
| 1 | VGG-16    | 138M | X | 78.98% |LR=1e-4, Batch_size=64, Epochs=60 | 92분 (1.5분/Epoch) |
| 2 | ResNet-50 | 25M | X | 86.93% |LR=1e-4, Batch_size=64, Epochs=60 | 57분 (1분/Epoch)|
| 3 | **EfficientNet-B0** | 5.3M | X | **85.61%** |LR=3e-4, Batch_size=64, Epochs=60 | 36분 (0.6분/Epoch)|
| 4 | VGG-16-bn | 138M | X | 89.71% |LRr=1e-4, Batch_size=64, Epochs=60 | 112분 (1.9분/Epoch)|
|||||||
| 5 | VGG-16    | 138M | O | 82.37% |LR=1e-4, Batch_size=64, Epochs=60 | 92분 (1.5분/Epoch)|
| 6 | ResNet-50 | 25M | O | 87.34% |LR=1e-4, Batch_size=64, Epochs=60 | 57분 (1분/Epoch)|
| 7 | **EfficientNet-B0** | 5.3M | O | **87.80%** |LR=3e-4, Batch_size=64, Epochs=60 | 36분 (0.6분/Epoch)|
| 8 | VGG-16-bn | 138M | O | 90.75% |LR=1e-4, Batch_size=64, Epochs=60 | 112분 (1.9분/Epoch)|

## 6. 구현 및 모델 학습 후기

EfficientNet-B0를 구현하고 학습을 진행했으나, 초기에는 기대한 만큼의 결과를 얻지 못했습니다.

- **(25.09 실험)**  
ResNet-50 대비 파라미터 수가 1/5 이고 (25.5M -> 5.3M) FLOP이 1/10 (4.1B -> 0.39B)인 점을 고려해 학습 시간이 크게 단축될 것으로 기대했습니다. 그러나 실제로는 epoch당 학습 시간이 약 10% 단축되는 데 그쳤습니다. 이후 모델 학습이 오래 걸리는 이유를 알아본 결과, Depthwise Convolution이 생각보다 연산 비용이 높다고 합니다. 이로 인해 학습 시간 단축이 기대한 만큼 되지 않은 것으로 이해하고 있습니다.

- **(26.01 실험)**  
DataLoader의 num_workers 설정을 조정한 이후 재학습을 진행한 결과, EfficientNet-B0의 epoch당 학습 시간은 0.6분, ResNet-50은 약 1분으로 측정되었습니다. 이를 통해 EfficientNet-B0가 ResNet-50 대비 학습 시간이 약 40% 단축되었음을 확인할 수 있었습니다. 25년 9월에 처음 구현하고 실험할 당시 모델이 가진 이점과 효율성을 충분히 끌어내기엔 제가 너무 미숙하고 기초가 부족했던 것 같습니다.

- **성능 비교**  
EfficientNet-B0 또한 ResNet-50과 마찬가지로 VGG16-bn보다 높은 성능을 보이지는 않았습니다. 비교적 작은 규모의 데이터셋을 사용한 환경에서 모델 복잡도에 따른 오버피팅의 영향을 받은 것으로 보입니다. 향후 ImagNet으로 사전 훈련된 모델을 파인튜닝하는 식으로 이들의 성능을 다시 비교해보려고 합니다. (구현6 레포지토리 참고)
