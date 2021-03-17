# install YOLO on Jetson TX2 with Ubuntu 18.04 LTS

This document is to help people struggling with installing YOLO on Python3 and Ubuntu 18.04.5 LTS.

Install the dependencies first, as specified in the below section. Then follow the rest of the guide.

### install jetpack to NVIDIA Jetson TX2

See in the "install-caffe"

### install

install from the stable version
```
git clone https://github.com/pjreddie/darknet
cd darknet
# Modify Makefile
make
```

Before running make, needs to modify Makefile to configure GPU/CPU.
```
GPU=1
CUDNN=1
CUDNN_HALF=0
OPENCV=1
AVX=0
OPENMP=0
LIBSO=0
ZED_CAMERA=0
ZED_CAMERA_v2_8=0

# Modify arch
ARCH=   -gencode arch=compute_30,code=sm_30 \
                -gencode arch=compute_35,code=sm_35 \
                -gencode arch=compute_50,code=sm_50 \
                -gencode arch=compute_52,code=sm_52 \
                -gencode arch=compute_60,code=sm_60 \
                -gencode arch=compute_61,code=sm_61 \
                -gencode arch=compute_61,code=compute_61

# Modify opencv to opencv4 if using opencv version >= 4.0
ifeq ($(OPENCV), 1)
COMMON+= -DOPENCV
CFLAGS+= -DOPENCV
LDFLAGS+= `pkg-config --libs opencv4` -lstdc++
COMMON+= `pkg-config --cflags opencv4`
endif
```

### Download weight file

```
# Use yolov3-tiny as an example
wget https://pjreddie.com/media/files/yolov3-tiny.weights
```

### Troubleshooting

1. Running without display

Use this fork

```
git clone https://github.com/AlexeyAB/darknet.git
cd darknet
```

Modify Makefile
```
GPU=1
CUDNN=1
CUDNN_HALF=0
OPENCV=1
AVX=0
OPENMP=0
LIBSO=0
ZED_CAMERA=0
ZED_CAMERA_v2_8=0

ARCH=   -gencode arch=compute_30,code=sm_30 \
                -gencode arch=compute_35,code=sm_35 \
                -gencode arch=compute_50,code=sm_50 \
                -gencode arch=compute_52,code=sm_52 \
                -gencode arch=compute_60,code=sm_60 \
                -gencode arch=compute_61,code=sm_61 \
                -gencode arch=compute_61,code=compute_61

# no need to modify opencv flag
```

Make sure that CUDA version >= 10.1, otherwise will cause compiling errors like this.

```
2 errors detected in the compilation of "/tmp/tmpxft_00003086_00000000-9_network_kernels.compute_70.cpp1.ii".
Makefile:185: recipe for target 'obj/network_kernels.o' failed
```

If using CUDA version == 10.0 and CUDNN version == 7.5, an early version is recommended.

```
git reset --hard 64efa721ede91cd8ccc18257f98eeba43b73a6a
```

Finally, add "-dont_show" to command.

```
# For example
./darknet detector demo cfg/coco.data cfg/yolo.cfg yolo.weights -dont_show test.mp4
```