# **Deploying bucketed SDXL image-to-image on inferencia2 instance**

## 1) 코드 작성의 배경
  
SDXL 모델은 1024x1024 픽셀로 입출력 이미지의 사이즈가 고정되어 있습니다. 이로 인해 1024x1024 pixel 사이즈보다 작은 이미지를 입력 프롬프트로 받게 되는 경우에도, 이미지에 패딩을 적용하여 1024x1024로 확대한 후 모델을 구동시켜, 패딩 영역에 대한 불필요한 연산을 수행해야만 합니다. 이는 컴퓨팅 리소스 및 동작 시간에 있어서 손실로 이어집니다. 버켓팅(Bucketing)은 SDXL 모델의 입출력 이미지의 사이즈를 몇단계로 구분하여, 다양한 사이즈의 입력 이미지에 대하여 가장 유사한 입출력 이미지를 갖는 SDXL 모델로 구동시키는 방법입니다. 이를 통해 입력 이미지에 대한 패딩 영역을 최소화할 수 있고, 이는 컴퓨팅 리소스 및 동작 시간 절감을 가능케 합니다. 이에 대한 자세한 설명은 다음 링크를 참고하시기 바랍니다.
https://awsdocs-neuron.readthedocs-hosted.com/en/latest/general/appnotes/torch-neuron/bucketing-app-note.html

## 2) 코드 범위
본 코드에서는 384x384 픽셀 사이즈, 512x512 픽셀 사이즈, 768x768 픽셀 사이즈, 896x896 픽셀 사이즈, 1024x1024 픽셀 사이즈의 입출력 이미지에 대하여 버켓팅된 다섯개 SDXL 모델을 컴파일하여 각각의 Neuron 모델을 생성하고, 이를 inferentia2 인스턴스에서 구동하는 방법을 설명합니다. 본 예제에서는 HuggingFace의 stabilityai/stable-diffusion-xl-refiner-1.0 모델과 EC2의 inf2.24xlarge 인스턴스를 사용하였습니다. 
  
## 3) 사전 환경 설정
시작에 앞서 Inf2 셋업 가이드에 따라 EC2 인스턴스에 torch-neuronx 와 neuronx-cc 를 설치해야합니다. 이에 관한 자세한 설명을 다음 링크를 참고하시기 바랍니다.
https://awsdocs-neuron.readthedocs-hosted.com/en/latest/general/setup/torch-neuronx.html#setup-torch-neuronx

## 4) 성능 요약
각각의 입출력 이미지를 대상으로 컴파일된 다섯 개 뉴런 모델은 동작 속도 측면에서 다음과 같은 성능을 보입니다.
384x384   --> 21.39it/sec||
512x512   --> 14.01it/sec ||
768x768   --> 5.16it/sec ||
896x896   --> 2.35it/sec ||
1024x1024 --> 2.28it/sec ||   
