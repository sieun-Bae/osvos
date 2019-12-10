https://eungbean.github.io/2019/07/03/OSVOS/

One-Shot Video Object Segmentation
S. Calles et al., CVPR 2017

190703-fig1 ***

Introduction
이 논문은 semi-supervised video object segmentation을 다룬 논문입니다.
video object segmentation(VOS)이란 비디오에서 찾고자 하는 물체를 배경으로부터 분리하는 문제입니다.

저자는 One-shot 방법을 적용하여 One-Shot Video Object Segmentation (OSVOS) 로 명명했습니다.
One-shot이란, 타겟을 한번 보면 (즉, 첫 프레임에서 찾고자 하는 물체를 마스크로 주어지면), 나머지 프레임에서 같은 물체를 찾아내는 방법입니다.
이 방법은 이후 많은 VOS 연구들에 영감을 주었습니다.

저자는 이 논문의 Contribution 을 다음과 같이 언급합니다.

One-Shot - 첫 프레임의 Object를 Segment해주기만 하면 나머지 프레임에서도 물체를 찾아낼 수 있음.
각 프레임을 독립적으로 연산함
기존에는 Temporal consistency를 중요하게 생각해서 급격하게 변하는 물체에 약한 모습을 보였는데, 딥러닝 (CNN)을 사용하면 각 프레임을 독립적으로 연산해도 Temporal Consistency를 얻을 수 있을 뿐만 아니라 Occlusion등에 강건한 장점을 가짐.
속도와 성능간의 Trade-off가 자유로워 유저 선택의 폭이 넓어짐.
이제, 논문을 자세히 살펴보겠습니다.

One-shot deep Learning
One-Shot이란, 학습 과정에서 모든 프레임에 대한 Groundtruth (GT)를 알려주는 대신, 첫 프레임의 GT를 알려주는 것 만으로도 나머지 프레임에서 물체를 학습할 수 있습니다.

사람이 물체를 인지할 때에는 두 단계의 사고를 하게 됩니다.

1) “이것은 물체이다.”
2) “이것이 그 물체이다.”

이에 착안하여, 저자는 네트워크를 두 단계로 학습시킵니다.

1) “이것은 물체이다.” : 배경으로부터 물체를 분리하도록 학습시킴. - Off-line
2) “이것이 그 물체이다.” : 분리된 물체 중 특정한 물체를 구별하도록 학습시킴. - On-line

사실 One-Shot의 개념은 조금 더 심오하고 흥미롭습니다. 각 클래스당 Training Data가 하나만 있어도 Classification을 잘 할 수 있다는 것으로, 얼굴인식등에서 널리 쓰인 개념입니다. 아래 자료를 보시면 이해에 도움이 되실겁니다.

One Shot Learning by Andrew Ng
One-Shot Learning by Siraj Raval
위에서 말하는 One-Shot learning과 여기서 말하는 One-shot의 개념이 어떻게 비슷하면서 다른지 비교해보는 것도 흥미롭습니다.

End-to-End trainable foreground FCN
1) VGG-Net을 기반으로,
2) 각 stage의 feature를 뽑아내어
3) 원래 이미지 사이즈로 Upscale한 뒤
4) 그 결과를 Linearly Fuse합니다.

이때 Loss Function은 다음과 같습니다.

L(W)=−∑j∈Y+logP(yj=1|X;dW)−∑j∈Y−logP(yj=0|X;W)
이는 Binary Classification을 위한 Pixel-wise cross entropy를 의미합니다. (Xie and Tu, 2015)
하지만, 이 경우 두 binary label의 imbalance 문제가 발생합니다. 이를 해결하기 위해서 수정한 Loss Function은 다음과 같습니다.

Lmod(W)=−β∑j∈Y+logP(yj=1|X;W)−(1−β)∑j∈Y−logP(yj=0|X;\boldW)
where β=|Y−|/|Y|
Training Details
190703-OSVOS-overall-network

저자는 VOS를 One-shot으로 수행하기 위해 3단계의 학습과정을 거쳤습니다.

1) Base Network - 영상의 feature를 학습
2) Parent Network - 영상에서 물체와 배경을 분리하는 방법을 학습
3) Test Network - 분리한 물체 중 원하는 물체만 추출하는 방법을 학습

1. Pre-Trained Network
가장 먼저, 네트워크가 영상에서 물체의 feature를 얻어낼 수 있어야 합니다. 이를 위해 ImageNet에 학습이 잘 된 FCN 네트워크를 가져와서 그냥 사용했습니다. Static Image로 먼저 학습을 시켰다고 보시면 됩니다.

2. Parent network
이제 네트워크는 Video를 학습하여 네트워크는 배경과 전경을 분리하는 방법을 배웁니다. 학습 데이터는 Davis Dataset의 Binary Mask를 학습합니다.
이는 필연적으로 비디오 전체의 시퀀스를 모두 input으로 넣어주고 output을 한번에 얻게 됩니다. 이를 Off-line Training 이라고 합니다. (반대로 각 프레임별로 연산하며 출력하는 것은 On-line 이라고 합니다.)

3. Test Network
이제 네트워크는 배경과 물체를 분리할 수 있습니다. 하지만 분리해낸 물체에는 원하지 않는 물체들도 섞여있습니다. 이렇게 분리해낸 여러가지 물체들 중, 첫 프레임에 주어진 물체만 골라서 분리하는 임무를 수행해야 합니다.
이제 One-Shot이 등장합니다. 첫 프레임에서 원하는 물체만 분리한 Binary Mask를 주는 것입니다. 그럼 네트워크는 첫 프레임의 Target Object Mask를 이용해서 나머지 프레임의 Target Object까지 정확하게 분리할 수 있게 됩니다.


이러한 과정을 fine-tuning 이라고 하고, 시간을 많이 할당할 수록 좋은 결과를 얻을 수 있습니다. (Trade-off)

Contour Snapping
190703-two-stream-FCN

영상에서 물체를 더욱 정확하게 찾기 위해, 저자는 Contour Snapping을 적용하였습니다. 앞서 설명했던 Foreground Branch만을 사용하게 되면, 물체의 윤곽선이 비교적 부정확한 것을 알 수 있습니다. 이를 보정하기 위해, 두번째 네트워크-Contour Branch 를 두어 다음과 같은 과정을 거칩니다.

1) 물체(Foreground)를 찾는 네트워크와
2) 윤곽선(Contour)을 찾는 네트워크를 학습시킨 후
3) 이 결과를 융합-Contour Snapping-하여 더욱 정확한 결과를 얻어냈습니다.

Contour Branch Network는 기본적으로 같은 구조를 가집니다. 다만, Foreground Branch가 GT Binary Mask를 학습했던 것과 달리
Contour Branch는 Contour를 학습 (PASCAL-Context Dataset)합니다.
두 네트워크에서 Foreground와 Contour를 얻었다면, 이를 합쳐(Snap)줍니다.

1) Foreground Mask에서 Superpixel을 계산합니다.
2) Ultrametric Contour Map (UCM)을 이용하여 Contour를 찾아줍니다. 3) Majority Voting을 통하여 Foreground Mask의 Superpixel을 선택합니다.

그 결과, Fast Bilateral Solver (FBS)를 이용하여 Snapping해준 방법보다 시간은 조금 더 걸려도 더욱 좋은 결과를 얻을 수 있었습니다.

Experimental Validation
Measures
VOS에서는 다음과 같은 세가지 metric으로 성능을 평가합니다.

IoU
Contour Accuracy
Temporal instability of the mask.
Results
190703-results [Ours]: original method [-BS]: without boundary snapping [-PN]: without pre-training the parent network on DAVIS [-OS]: without performing the one-shot learning on the specific sequence. 작은 숫자는 Original method에 대한 성능 차이를 의미합니다.

Ablation Study
다른 방법에 비해서도 결과가 좋았다.
Parent Network에서 학습 Image가 많을 수록 좋았지만, 200장정도에서 Saturation되더라.
Fine-tuning에서 시간을 오래 부여할 수록 결과가 좋았지만, Saturation 되더라.
Annotation Frame을 더 많이 줄수록 좋았지만, 두 장 정도면 충분하더라.
Tracking도 잘 되더라.
끝. 부족한 부분이나 틀린 부분은 댓글로 달아주시면 감사드리겠습니다.
