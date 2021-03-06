# Darknet Train using Docker

<!--
```bash
$ nvidia-smi
Sun Aug  8 00:00:00 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.100      Driver Version: 440.100      CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 208...  Off  | 00000000:07:00.0  On |                  N/A |
|  0%   46C    P8    13W / 300W |    329MiB / 11016MiB |      3%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce RTX 208...  Off  | 00000000:08:00.0 Off |                  N/A |
|  0%   45C    P8    19W / 300W |      1MiB / 11019MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1257      G   /usr/lib/xorg/Xorg                           159MiB |
|    0      1620      G   /usr/bin/gnome-shell                         123MiB |
|    0      3195      G   ...quest-channel-token=1242754644392399813    43MiB |
+-----------------------------------------------------------------------------+
```
-->

## Build using Docker

### [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda)

```bash
docker pull nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
```

```bash
$ docker run --gpus all nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04 nvidia-smi
Sun Aug  8 00:00:00 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.100      Driver Version: 440.100      CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 208...  Off  | 00000000:07:00.0  On |                  N/A |
|  0%   48C    P8    14W / 300W |    340MiB / 11016MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce RTX 208...  Off  | 00000000:08:00.0 Off |                  N/A |
|  0%   45C    P8    19W / 300W |      1MiB / 11019MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

Why using CUDA 10.2 here:

1. nvidia-smi, CUDA Version: 10.2
2. PyTorch 1.6.0, CUDA 10.2

<!--
docker run -it --gpus all nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
-->

### [OpenCV](https://github.com/opencv/opencv)

```bash
cd docker/ubuntu18.04-cuda10.2/opencv4.4.0/

docker build \
-t joinaero/ubuntu18.04-cuda10.2:opencv4.4.0 \
--build-arg opencv_ver=4.4.0 \
--build-arg opencv_url=https://gitee.com/cubone/opencv.git \
--build-arg opencv_contrib_url=https://gitee.com/cubone/opencv_contrib.git \
.
```

<!--
docker run -it --gpus all joinaero/ubuntu18.04-cuda10.2:opencv4.4.0
-->

<!--
#9 99.47 W: GPG error: https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu1804/x86_64  Release: The following signatures were invalid: BADSIG F60F4B3D7FA2AF80 cudatools <cudatools@nvidia.com>
#9 99.47 E: The repository 'https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64  Release' is not signed.
-->
<!--
#9 1398. [ 63%] Building CXX object modules/xfeatures2d/CMakeFiles/opencv_xfeatures2d.dir/src/boostdesc.cpp.o
#9 1398. /codes/opencv_contrib/modules/xfeatures2d/src/boostdesc.cpp:654:20: fatal error: boostdesc_bgm.i: No such file or directory
#9 1398.            #include "boostdesc_bgm.i"
-->

### [Darknet](https://github.com/AlexeyAB/darknet)

```bash
cd docker/ubuntu18.04-cuda10.2/opencv4.4.0/darknet/

docker build \
-t joinaero/ubuntu18.04-cuda10.2:opencv4.4.0-darknet \
.
```

<!--
#9 147.6 /usr/bin/ld: warning: libcuda.so.1, needed by libdarknet.so, not found (try using -rpath or -rpath-link)
#9 147.6 libdarknet.so: undefined reference to `cuCtxGetCurrent'
#9 147.6 collect2: error: ld returned 1 exit status
-->

## How to detect image with pre-trained models

* Pre-trained models: [yolov4.weights](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.weights)
* Datasets: [MS COCO](http://cocodataset.org/#download)

```bash
xhost +local:docker

docker run -it --gpus all \
-e DISPLAY \
-e QT_X11_NO_MITSHM=1 \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-v $HOME/.Xauthority:/root/.Xauthority \
--name darknet \
--mount type=bind,source=$HOME/Codes/devel/datasets/coco2017,target=/home/coco2017 \
--mount type=bind,source=$HOME/Codes/devel/models/yolov4,target=/home/yolov4 \
joinaero/ubuntu18.04-cuda10.2:opencv4.4.0-darknet
```

```bash
./darknet detector test cfg/coco.data cfg/yolov4.cfg /home/yolov4/yolov4.weights \
-thresh 0.25 -ext_output -show -out /home/coco2017/result.json \
/home/coco2017/test2017/000000000001.jpg
```

![](images/coco2017-test2017-000000000001.png)

<!--
CUDA status Error: file: /home/darknet/src/dark_cuda.c : () : line: 39 : build time: Aug 10 2020 - 00:00:00

 CUDA Error: forward compatibility was attempted on non supported HW
CUDA Error: forward compatibility was attempted on non supported HW: Operation not permitted
-->

## How to train on MS COCO 2017 dataset

> Train a subset of objects on MS COCO 2017 dataset.

### Train Detector

#### Required files

* [cfg/coco/coco.names](../cfg/coco/coco.names) &lt;[cfg/coco/coco.names.bak](../cfg/coco/coco.names.bak) has original 80 objects&gt;
  * Edit: keep desired objects
* [cfg/coco/yolov4.cfg](../cfg/coco/yolov4.cfg) &lt;[cfg/coco/yolov4.cfg.bak](../cfg/coco/yolov4.cfg.bak) is original file&gt;
  * Download [yolov4.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov4.cfg), then changed:
  * `batch`=64, `subdivisions`=32 &lt;32 for 8-12 GB GPU-VRAM&gt;
  * `width`=512, `height`=512 &lt;any value multiple of 32&gt;
  * `classes`=&lt;your number of objects in each of 3 [yolo]-layers&gt;
  * `max_batches`=&lt;classes\*2000, but not less than number of training images and not less than 6000&gt;
  * `steps`=&lt;max_batches\*0.8, max_batches\*0.9&gt;
  * `filters`=&lt;(classes+5)x3, in the 3 [convolutional] before each [yolo] layer&gt;
  * <s>`filters`=&lt;(classes+9)x3, in the 3 [convolutional] before each [Gaussian_yolo] layer&gt;</s>
* [cfg/coco/coco.data](../cfg/coco/coco.data)
  * Edit: `train`, `valid` to YOLO datas
* csdarknet53-omega.conv.105
  * Download [csdarknet53-omega_final.weights](https://drive.google.com/open?id=18jCwaL4SJ-jOvXrZNGHJ5yz44g9zi8Hm), then run:

  ```bash
  docker run -it --rm --gpus all \
  --mount type=bind,source=$HOME/Codes/devel/models/yolov4,target=/home/yolov4 \
  joinaero/ubuntu18.04-cuda10.2:opencv4.4.0-darknet

  ./darknet partial cfg/csdarknet53-omega.cfg /home/yolov4/csdarknet53-omega_final.weights /home/yolov4/csdarknet53-omega.conv.105 105
  ```

#### YOLO datas

```bash
cd start-yolov4/
pip install -r scripts/requirements.txt

export COCO_DIR=$HOME/Codes/devel/datasets/coco2017

# train
python scripts/coco2yolo.py \
--coco_img_dir $COCO_DIR/train2017/ \
--coco_ann_file $COCO_DIR/annotations/instances_train2017.json \
--yolo_names_file ./cfg/coco/coco.names \
--output_dir ~/yolov4/coco2017/ \
--output_name train2017 \
--output_img_prefix /home/yolov4/coco2017/train2017/

# valid
python scripts/coco2yolo.py \
--coco_img_dir $COCO_DIR/val2017/ \
--coco_ann_file $COCO_DIR/annotations/instances_val2017.json \
--yolo_names_file ./cfg/coco/coco.names \
--output_dir ~/yolov4/coco2017/ \
--output_name val2017 \
--output_img_prefix /home/yolov4/coco2017/val2017/
```

Result:

```txt
~/yolov4/coco2017/
├── train2017/
│   ├── 000000000071.jpg
│   ├── 000000000071.txt
│   ├── ...
│   ├── 000000581899.jpg
│   └── 000000581899.txt
├── train2017.txt
├── val2017/
│   ├── 000000001353.jpg
│   ├── 000000001353.txt
│   ├── ...
│   ├── 000000579818.jpg
│   └── 000000579818.txt
└── val2017.txt
```

#### Training

```bash
cd start-yolov4/

xhost +local:docker

docker run -it --gpus all \
-e DISPLAY \
-e QT_X11_NO_MITSHM=1 \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-v $HOME/.Xauthority:/root/.Xauthority \
--name darknet \
--mount type=bind,source=$HOME/Codes/devel/models/yolov4,target=/home/yolov4 \
--mount type=bind,source=$HOME/yolov4/coco2017,target=/home/yolov4/coco2017 \
--mount type=bind,source=$PWD/cfg/coco,target=/home/cfg \
joinaero/ubuntu18.04-cuda10.2:opencv4.4.0-darknet


mkdir -p /home/yolov4/coco2017/backup

# Training command
./darknet detector train /home/cfg/coco.data /home/cfg/yolov4.cfg /home/yolov4/csdarknet53-omega.conv.105

# Continue training
./darknet detector train /home/cfg/coco.data /home/cfg/yolov4.cfg /home/yolov4/coco2017/backup/yolov4_last.weights

# How to train with multi-GPU
# 1. Train it first on 1 GPU for like 1000 iterations
# 2. Then stop and by using partially-trained model `/backup/yolov4_1000.weights` run training with multigpu
./darknet detector train /home/cfg/coco.data /home/cfg/yolov4.cfg /home/yolov4/coco2017/backup/yolov4_1000.weights -gpus 0,1
```

> Train with flag `-map` then you will see mAP indicator `Last accuracy mAP@0.5 = 18.50%` in the console - this indicator is better than Loss, so train while mAP increases.

![](images/coco2017-train-chart.png)

#### Detection

```bash
./darknet detector test /home/cfg/coco.data /home/cfg/yolov4.cfg /home/yolov4/coco2017/backup/yolov4_final.weights \
-ext_output -show /home/yolov4/coco2017/val2017/000000006040.jpg
```

![](images/coco2017-val2017-000000006040.png)

To check accuracy mAP@IoU=50:

```bash
$ ./darknet detector map /home/cfg/coco.data /home/cfg/yolov4.cfg /home/yolov4/coco2017/backup/yolov4_final.weights
...
Loading weights from /home/yolov4/coco2017/backup/yolov4_final.weights...
 seen 64, trained: 384 K-images (6 Kilo-batches_64)
Done! Loaded 162 layers from weights-file

 calculation mAP (mean average precision)...
 Detection layer: 139 - type = 27
 Detection layer: 150 - type = 27
 Detection layer: 161 - type = 27
160
 detections_count = 745, unique_truth_count = 190
class_id = 0, name = train, ap = 80.61%   	 (TP = 142, FP = 18)

 for conf_thresh = 0.25, precision = 0.89, recall = 0.75, F1-score = 0.81
 for conf_thresh = 0.25, TP = 142, FP = 18, FN = 48, average IoU = 75.31 %

 IoU threshold = 50 %, used Area-Under-Curve for each unique Recall
 mean average precision (mAP@0.50) = 0.806070, or 80.61 %
Total Detection Time: 4 Seconds
```

<!--
### Evaluate Accuracy and FPS
-->

## References

* [Train Detector on MS COCO (trainvalno5k 2014) dataset](https://github.com/AlexeyAB/darknet/wiki/Train-Detector-on-MS-COCO-(trainvalno5k-2014)-dataset)
* [How to evaluate accuracy and speed of YOLOv4](https://github.com/AlexeyAB/darknet/wiki/How-to-evaluate-accuracy-and-speed-of-YOLOv4)
* [How to train (to detect your custom objects)](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects)

<!--
* [Pytorch搭建YoloV4目标检测平台](https://blog.csdn.net/weixin_44791964/article/details/106214657)
-->
