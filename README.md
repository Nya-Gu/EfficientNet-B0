# **1. EfficientNet-B0 구현**

## **1-1. EfficientNet-B0를 구현하게 된 동기**
다양한 학습 전략을 시도하고 싶었고, ResNet-50보다 가벼우면서 성능은 그에 뒤지지 않는 모델을 찾다가 발견한 것이 EfficientNet이었습니다. 전에 위클리 미션에서 이름을 들어본 모델이었기에 이번에 구현하기로 했습니다.

## **1-2. 구현에 참고한 논문**
**[1]   EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks**
([논문 링크](https://arxiv.org/abs/1905.11946))

이름 그대로 EfficientNet을 소개한 논문입니다. Neural Architecture Search(NAS)로 최적의 딥러닝 모델을 찾아내고, depth, width, resolution을 체계적이고 종합적으로 스케일링해 모델 성능을 향상시킨 점이 인상 깊었습니다. EfficientNet-B0부터 B7까지 다양한 모델을 제시하며 타 모델들과 이미지 분류 정확도, 추론 시간, 파라미터 수, FLOP 등을 비교하는데 테이블과 그래프 시각화가 잘 되어 있어 이해하기 편했습니다. 멋진 논문이었습니다.

**[2] MnasNet: Platform-Aware Neural Architecture Search for Mobile** ([논문 링크](https://arxiv.org/abs/1807.11626))

저자들이 EfficientNet 모델 패밀리의 백본이 되는 EfficientNet-B0를 어떻게 설계한 걸까요? 바로 이 논문에서 제안한 방법론을 활용했다고 합니다. EfficientNet 논문에서도 해당 방법론에 대해 간략히 언급하지만 더 자세한 내용이 궁금하시면 이 논문을 보면 될 것 같습니다. EfficientNet의 MBConv Block이 궁금하신 분들도 이 논문을 참고하시면 될 것 같습니다.

**[3] MobileNetV2: Inverted Residuals and Linear Bottlenecks** ([논문 링크](https://arxiv.org/abs/1801.04381))

Inverted Bottleneck Residual block을 공부할 때 참고한 논문입니다. 2번 논문과 함께 구현에 많은 도움이 되었습니다. 초반 내용이 잘 이해되진 않았습니다. 읽으면서 아직 공부가 더 필요한 것 같다고 느꼈습니다.

**[4] Squeeze-and-Excitation Networks** ([논문 링크](https://arxiv.org/abs/1709.01507))

위 논문을 참고해 SE Block을 구현했습니다. 메커니즘이 약간 Attention과 유사해서 흥미로웠습니다. Attention에서 각 입력 간 관계를 학습하고 가중치를 주는 것처럼 SE Block에서도 그와 유사하게 입력 피처 맵의 통계량 간 관계를 학습하고 이를 기반으로 가중치를 계산하는 과정이 있었습니다. 읽으면서 '만약 CNN 전용 Attention이 있다면 이런 느낌이지 않을까?' 하는 상상을 했습니다. 재밌는 논문이었습니다.

## **1-3. 구현 및 학습 실험 후기**
열심히 구현했으나 기대한 만큼 결과가 나오지 않아 아쉬웠습니다.
* (25년) ResNet-50과 비교해서 파라미터 수가 1/5 이고 (26M >> 5.3M) FLOP도 1/10 이어서 (4.1B >> 0.39B) 학습 시간이 배로 단축되지 않을까 기대했습니다. 하지만 안타깝게도 그런 일은 없었습니다. 1에폭당 학습 시간이 10% 단축되는 데 그쳤습니다. 생각보다 학습이 오래 걸리는 이유를 알아보니 Depthwise Conv을 계산하는 데 시간이 오래 걸린다고 합니다. 이번에 실험하면서 새로 알았습니다.

* (26년) num_workers 설정 이후 재학습한 결과, EfficientNet-B0의 1에폭당 학습 시간은 0.6분, ResNet-50의 1에폭당 학습 시간은 1분으로 EfficientNet-B0가 ResNet-50보다 학습 시간이 약 40% 줄어든 것을 확인할 수 있었습니다. EfficientNet을 처음 구현하고 학습시킬 당시 모델의 이점을 다 끌어내기엔 제가 기초가 너무 없었던 것 같습니다.

* EfficientNet-B0도 ResNet-50과 마찬가지로 VGG16-bn보다 성능이 좋게 나오진 못했습니다. 학습 데이터셋이 작아서 그런 게 아닐까 추측하고 있습니다. 차후 ImageNet으로 사전훈련된 모델을 파인튜닝하는 방식으로 이들의 성능을 다시 비교해보려고 합니다. (구현6 참고)

## **1-4. 모델 구현**
Pytorch에 구현된 모델과 동일한 구조, 동일한 파라미터 수를 가지는 EfficientNet-B0을 구현했습니다.  
[구현3] EfficientNet-B0.ipynb 파일 참고 바랍니다.  

# **5. 학습 결과 (25.09)**
여러 차례 실험 후 가장 높게 나온 성능을 기록했습니다. (+ 다른 모델의 학습 결과도 비교를 위해 가져와봤습니다.)  
[적용한 데이터 증강]
- RandomHorizontalFlip(p=0.5)
- ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.05)

| 실험 모델 | 데이터 증강 여부 | Validation Accuracy | 학습 상세 | 학습 시간 (L4 GPU) |
|-----------|:----------------:|:-------------------:|-----------|-----------|
| VGG-16-bn | X | 83.2% |lr=1e-4, batch_size=64, epochs=20| 60분 (3분/epoch) |
| VGG-16-bn | O | 86.4% |lr=1e-4, batch_size=64, epochs=14| 190분 (13.5분/epoch) |
| ResNet-50 | X | 81.7% |lr=2e-4, batch_size=32, epochs=10| 20분 (2분/epoch) |
| ResNet-50 | O | 83.0% |lr=1e-4, batch_size=32, epochs=15| 170분 (11.5분/epoch) |
| EfficientNet-B0 | X | 78.6% |lr=1e-3, batch_size=32, epochs=20| 35분 (1.75분/epoch) |
| EfficientNet-B0 | O | 83.2% |lr=1e-3, batch_size=64, epochs=14| 140분 (10분/epoch) |

# **6. 재학습 결과 (26.01)**
[적용한 데이터 증강]
- RandomHorizontalFlip(p=0.5)
- ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.05)

| No. | 실험 모델 | 데이터 증강 여부 | Validation Accuracy | 학습 상세 | 학습 시간 (L4 GPU) |
|-----|-----------|:----------------:|:-------------------:|-----------|-----------|
| 1 | VGG-16    | X | 78.98% |lr=1e-4, batch_size=64, epochs=60 | 92분 (1.5분/epoch) |
| 2 | VGG-16    | O | 82.37% |lr=1e-4, batch_size=64, epochs=60 | 92분 (1.5분/epoch)|
| 3 | ResNet-50 | X | 86.93% |lr=1e-4, batch_size=64, epochs=60 | 57분 (1분/epoch)|
| 4 | ResNet-50 | O | 87.34% |lr=1e-4, batch_size=64, epochs=60 | 57분 (1분/epoch)|
| 5 | **EfficientNet-B0** | X | **85.61%** |lr=3e-4, batch_size=64, epochs=60 | 36분 (0.6분/epoch)|
| 6 | **EfficientNet-B0** | O | **87.80%** |lr=3e-4, batch_size=64, epochs=60 | 36분 (0.6분/epoch)|
| 7 | VGG-16-bn | X | 89.71% |lr=1e-4, batch_size=64, epochs=60 | 112분 (1.9분/epoch)|
| 8 | VGG-16-bn | O | 90.75% |lr=1e-4, batch_size=64, epochs=60 | 112분 (1.9분/epoch)|

증강 여부를 기준으로 정렬하면 다음과 같습니다.

| No. | 실험 모델 | 데이터 증강 여부 | Validation Accuracy | 학습 상세 | 학습 시간 (L4 GPU) |
|-----|-----------|:----------------:|:-------------------:|-----------|-----------|
| 1 | VGG-16    | X | 78.98%               |lr=1e-4, batch_size=64, epochs=60 | 92분 (1.5분/epoch) |
| 2 | ResNet-50 | X | 86.93%               |lr=1e-4, batch_size=64, epochs=60 | 57분 (1분/epoch)|
| 3 | **EfficientNet-B0** | X | **85.61%** |lr=3e-4, batch_size=64, epochs=60 | 36분 (0.6분/epoch)|
| 4 | VGG-16-bn | X | 89.71%               |lr=1e-4, batch_size=64, epochs=60 | 112분 (1.9분/epoch)|
|||||||
| 5 | VGG-16    | O | 82.37%               |lr=1e-4, batch_size=64, epochs=60 | 92분 (1.5분/epoch)|
| 6 | ResNet-50 | O | 87.34%               |lr=1e-4, batch_size=64, epochs=60 | 57분 (1분/epoch)|
| 7 | **EfficientNet-B0** | O | **87.80%** |lr=3e-4, batch_size=64, epochs=60 | 36분 (0.6분/epoch)|
| 8 | VGG-16-bn | O | 90.75%               |lr=1e-4, batch_size=64, epochs=60 | 112분 (1.9분/epoch)|
