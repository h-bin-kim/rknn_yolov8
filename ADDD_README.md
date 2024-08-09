# End2End rknn model exportation

## Source
Based on repositories below.
- https://github.com/ultralytics/ultralytics
- https://github.com/airockchip/ultralytics_yolov8


## What different from [Rockchip official Repo](https://github.com/airockchip/ultralytics_yolov8)
The Python post-processing code (dfl) from Rockchip, executed on the CPU, is still slow even on the CPU.

- https://github.com/airockchip/rknn_model_zoo/blob/main/examples/yolov8/python/yolov8.py
- When running the above code on our SBC, we measured that about 60% of the post-processing time was spent on the exponential operation of dfl.
  - This is the same whether using the dfl (Torch-based) or a custom dfl code based on Numpy.
- There may be instances where we need to leave some CPU headroom for other more important CPU-bound tasks.

Additionally, the numerous model output branches felt complex. We wanted to create a model as close to End2End as possible.

To summarize the changes:

- We removed the code that alters the output values at the detection model output node (Head) when using the rknn format.
- We changed the split method of the detection model output node to a slice method, which is used in the edgetpu format export.
  - (It's unclear if this has any significance since we merely added rknn to the options list.)
- **[Important]** We added an rknn format option to the bbox output coordinate normalization branch.

Our ultimate goal is to successfully convert the Pose model. Although we faced some challenges, we were able to solve it more easily than expected. 
- The most problematic part of the E2E rknn model conversion process was the normalization method. 
- By slightly modifying the kpts_decode function in the pose head class, we were able to successfully confirm the int8 Quantization results.

All these changes can be checked in the commit history.

With this method, the converted model can reuse familiar post-processing codes from Ultralytics.

## How to use
- Build the python environment required to use ultralytics and rknn-toolkit.
- Clone this repository and install the package in the virtual environment with the following command from the repository path:

    ```pip install .```
- Try model conversion with the following command:

    ex) ```yolo export model=yolov8n-pose.pt format='rknn' imgsz=[640,640]```


## Convert to RKNN model, Python demo, C demo

Please refer to https://github.com/airockchip/rknn_model_zoo.