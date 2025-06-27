# yolov5_widerface
YoloV5n and YOloV5s from v6.2 were trained on widerface dataset for 100 epochs and then evaluated on FDDB dataset using its official evaluator. Python 3.6.9, PyTorch 1.10, OpenCV 4.8.0, Jetpack 4.6.1

## TensorRT Inference
CUDA 10.2 and TensorRT 8.2.1.8
The models were converted to ONNX using YoloV5 official repository `python3 export.py --batch 1 --include onnx --simplify --data data/wider_face.yaml --weights weights/yolov5s-face.pt --opset 12 --conf-thres 0.45  --imgsz 640 640`.

The ONNX file was converted to TensorRT on Jetson Nano 2GB on both FP32 and FP16. 2GB variant does not support INT8 data type
```
trtexec --onnx=yolov5n-face.onnx --saveEngine=yolov5n-face.engine
trtexec --onnx=yolov5n-face.onnx --saveEngine=yolov5n-face.engine --fp16
```
Run the model using one of these sample scripts:
- https://github.com/jario-jin/yolov5-on-nvidia-jetson
- https://github.com/alxmamaev/jetson_yolov5_tensorrt

## YoloV5 on DeepStream 6.0.1
Generate ONNX file using deepstream yolov5 export [script](https://github.com/marcoslucianops/DeepStream-Yolo/blob/master/docs/YOLOv5.md). This step can be done on any machine (does not need to be on Jetson Nano) `python3 export_yoloV5.py -w yolov5s.pt --simplify`. The script might fail to generate a labels file and face an error. If this happens, just skip the labels generation part and manually create the labels file. The line number is the index and the string on the line if the label name.

Copy the file ONNX file to DeepStream-YOLO directory. Edit the yolo config file (in this case yolov5).

To capture live feed from USB camera change the source in yolo config file.
```
[source0]
type=1
camera-width=640
camera-height=640
camera-fps-n=30
camera-v4l2-dev-node=0
```

Check what resolutions are supported by the USB camera using `v4l-utils`
```
# Install the video utility "v4l-utils"
sudo apt-get install v4l-utils -y

# List the camera properties e.g. the supported reolutions etc
v4l2-ctl -d 0 --all
```
Update `deepstream_app_config.txt` to use yolov5 configuration file. Compile the libraries and run `deepstream-app -c deepstream_app_config.txt`. The generated TensorRT files have a common name be defailt. Either rename it after generation or edit the cpp file and recompile them.

## FDDB
To generate the discrete and continuous curves use one of the docker images
- https://hub.docker.com/r/tahirishaq10/fddb_evaluator
- https://hub.docker.com/r/housebw/fddb-evaluator

Make sure to follow the file name and directory structure otherwise, results will be generated.

The official visualization software uses GNUplot to generate the plots. Some have implemented it using python.

## References
- [Converting Widerface to YoloV5 format](https://github.com/LambdaLabsML/examples/tree/main/yolov5)
- [YoloV5 widerface training blog](http://lambda.ai/blog/training-a-yolov5-object-detector-on-lambda-cloud)
- [DeepStream USB Camera](https://forums.developer.nvidia.com/t/deepstream-4-deepstream-app-yolov2-tiny-with-video-stream-from-camera/78743)
- [Increase YOLO performace in DeepStream](https://forums.developer.nvidia.com/t/deepstream-6-yolo-performance-issue/194238/20)
- [DeepStream Examples - v1.1.1 is for deepstream 6.0.1](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps)
- [Intro to DeepStream](https://blog.ml6.eu/nvidia-deepstream-quickstart-9147dd49a15d)