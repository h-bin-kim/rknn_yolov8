# End2End rknn 모델 변환

## 출처
아래 레포지토리에 기반합니다.
- https://github.com/ultralytics/ultralytics
- https://github.com/airockchip/ultralytics_yolov8


## 변경점 vs [Rockchip official Repo](https://github.com/airockchip/ultralytics_yolov8)
Rockchip에서 제공하는 공식 모델 변환 코드에서는 dfl이 npu에서 느리기 때문에, 모델에 변환과정에서 제외한다고 되어있습니다. 

이렇게 제외된 dfl 및 후처리 코드는 따로 작성되어 rknn 런타임으로 실행되는 단계에 붙여지게 됩니다. 이 과정에서 모델 출력이 너무 많은 분기를 가지게되고, 후처리 코드를 재활용할 수 없게되며, 프로젝트가 복잡해진다고 여겼습니다. 

- https://github.com/airockchip/rknn_model_zoo/blob/main/examples/yolov8/python/yolov8.py
- 보유한 SBC에서 위 코드를 실행했을때 후처리시간의 약 60%정도가 dfl의 exponential 연산에 소요되는것으로 측정했습니다. 
  - 위 코드의 dfl(torch 기반) 및 numpy 기반의 dfl 커스텀 코드로 실행했을 때에도 동일합니다. 
- 필요에 따라 더 중요한 다른 CPU-bound task를 위해 cpu에 여유를 남겨두어야할 경우가 있습니다. 그리고 NPU에서 그렇게 비효율적인 것으로 보이지도 않습니다. 몇 달 사이에 RKNN SDK가 개선된 것인지는 불분명합니다.  

변경사항을 요약하면 다음과 같습니다.:

- Detection 모델 출력노드(Head)에서 rknn 포맷일 경우 출력값을 변경하는 코드를 제거했습니다. 
- Detection 모델 출력노드의 split 방식을 edgetpu 포맷 방식 export에서 사용하는 slice 방식으로 변경했습니다. 
  - (그저 옵션 목록에 rknn을 추가한 것이기 때문에 의미가 있는지는 모릅니다.)
- **[중요]** bbox 출력 좌표 정규화 분기에 rknn 포맷 옵션을 추가했습니다. 

우리를 최종 목적은 Pose 모델을 성공적으로 변환하는 것 입니다. 
- E2E로 rknn 모델 변환을 하는 과정에서 가장 문제가 된 부분은 정규화를 하는 방식이었습니다. 
- pose head 클래스에 있는 kpts_decode 함수를 일부 수정해서 정상적으로 int8 Quantization 결과를 확인했습니다. 


이 모든 변경사항은 커밋 기록에서 확인할 수 있습니다. 

이 방식으로 변환된 모델은 익숙한 Ultralytics의 후처리 코드들을 재활용할 수 있습니다. 

## How to use
- ultralytics 및 rknn-toolkit을 사용할 때 필요한 python 환경을 구축합니다.
- 이 레포지토리를 clone하고, 레포지토리 경로에서 다음 명령어를 통해 가상환경에 패키지를 설치합니다. 

    ```pip install .```
- 다음 명령어를 통해 모델 변환을 시도합니다. 

    ex) ```yolo export model=yolov8n-pose.pt format='rknn' imgsz=[640,640]```


## Convert to RKNN model, Python demo, C demo

Please refer to https://github.com/airockchip/rknn_model_zoo.