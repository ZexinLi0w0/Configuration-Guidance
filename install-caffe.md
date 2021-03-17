# install caffe with python3 on ubuntu 18.04

This document is to help people struggling with installing Caffe on Python3 and Ubuntu 18.04.5 LTS.

Based on https://github.com/dungba88/caffe-python3-install.

Install the dependencies first, as specified in the below section. Then follow the rest of the guide.

### install jetpack to NVIDIA Jetson TX2

```shell
sudo apt-get install nvidia-jetpack
sudo apt-get install nsight-graphics-for-l4t cuda-toolkit-10-2 cuda-cross-aarch64-10-2 libopencv libopencv-dev opencv-licenses libopencv-python libopencv-samples libvisionworks libvisionworks-dev libvisionworks-samples libvisionworks-sfm libvisionworks-sfm-dev libvisionworks-tracking libvisionworks-tracking-dev vpi vpi-dev vpi-samples nsight-systems-2020.2.3
```

This operation may require 7GB+ disk space.

### install dependencies

```shell
# python3 modules (numpy, protobuf, skimage)
sudo pip3 install numpy
sudo apt-get install python3-skimage
sudo apt-get install python3-protobuf

# build essential
sudo apt-get install build-essential cmake git pkg-config

# gflags, glog, lmdb
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev

# boost
sudo apt-get install libboost-all-dev

# hdf5
sudo apt-get install libhdf5-dev

# protobuf
sudo apt-get install protobuf-compiler libprotobuf-dev

# blas
sudo apt-get install libblas-dev libcblas-dev libatlas-base-dev libopenblas-dev
sudo apt-get install libatlas-base-dev

# leveldb
sudo apt-get install libleveldb-dev

# snappy
sudo apt-get install libsnappy-dev

# others
# opencv dev
sudo apt-get install libopencv-dev
```

### install opencv3

Guide is taken from http://www.pyimagesearch.com/2015/07/20/install-opencv-3-0-and-python-3-4-on-ubuntu/

1. Checkout and compile

```shell
git clone https://github.com/opencv/opencv
cd opencv
mkdir build
cd build
cmake ..
make
sudo make install
```

### install caffe (with GPU support)

1. First check CUDA and CUDNN version (default installed by nvidia-jetpack)

```shell
# check CUDA version
nvcc -V
# check CUDNN version
cat /usr/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

Note that the default installed CUDA version is 10.2, and CUDNN version is 8.0 (on 2021/3/8), which may cause errors in the following section. CUDA version == 10.0 and CUDNN version == 7.5 is recommended. (It can be installed by Jetpack 4.2 by host installer)

2. Modify config file (the most important part)

```shell
cd ~
git clone https://github.com/BVLC/caffe
cd caffe
cp Makefile.config.example Makefile.config
```

Makefile.config

```
## Refer to http://caffe.berkeleyvision.org/installation.html
# Contributions simplifying and improving our build system are welcome!

# cuDNN acceleration switch (uncomment to build with cuDNN).
USE_CUDNN := 1

# CPU-only switch (uncomment to build without GPU support).
# CPU_ONLY := 1

# uncomment to disable IO dependencies and corresponding data layers
# USE_OPENCV := 0
# USE_LEVELDB := 0
# USE_LMDB := 0
# This code is taken from https://github.com/sh1r0/caffe-android-lib
# USE_HDF5 := 0

# uncomment to allow MDB_NOLOCK when reading LMDB files (only if necessary)
#	You should not set this flag if you will be reading LMDBs with any
#	possibility of simultaneous read and write
# ALLOW_LMDB_NOLOCK := 1

# Uncomment if you're using OpenCV 3
OPENCV_VERSION := 4

# To customize your choice of compiler, uncomment and set the following.
# N.B. the default for Linux is g++ and the default for OSX is clang++
# CUSTOM_CXX := g++

# CUDA directory contains bin/ and lib/ directories that we need.
CUDA_DIR := /usr/local/cuda
# On Ubuntu 14.04, if cuda tools are installed via
# "sudo apt-get install nvidia-cuda-toolkit" then use this instead:
# CUDA_DIR := /usr

# CUDA architecture setting: going with all of them.
# For CUDA < 6.0, comment the *_50 through *_61 lines for compatibility.
# For CUDA < 8.0, comment the *_60 and *_61 lines for compatibility.
# For CUDA >= 9.0, comment the *_20 and *_21 lines for compatibility.
# CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
		-gencode arch=compute_20,code=sm_21 \
		
CUDA_ARCH := -gencode arch=compute_30,code=sm_30 \
		-gencode arch=compute_35,code=sm_35 \
		-gencode arch=compute_50,code=sm_50 \
		-gencode arch=compute_52,code=sm_52 \
		-gencode arch=compute_60,code=sm_60 \
		-gencode arch=compute_61,code=sm_61 \
		-gencode arch=compute_61,code=compute_61

# BLAS choice:
# atlas for ATLAS (default)
# mkl for MKL
# open for OpenBlas
BLAS := atlas
# Custom (MKL/ATLAS/OpenBLAS) include and lib directories.
# Leave commented to accept the defaults for your choice of BLAS
# (which should work)!
# BLAS_INCLUDE := /path/to/your/blas
# BLAS_LIB := /path/to/your/blas

# Homebrew puts openblas in a directory that is not on the standard search path
# BLAS_INCLUDE := $(shell brew --prefix openblas)/include
# BLAS_LIB := $(shell brew --prefix openblas)/lib

# This is required only if you will compile the matlab interface.
# MATLAB directory should contain the mex binary in /bin.
# MATLAB_DIR := /usr/local
# MATLAB_DIR := /Applications/MATLAB_R2012b.app

# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
# PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/lib/python2.7/dist-packages/numpy/core/include
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
# ANACONDA_HOME := $(HOME)/anaconda
# PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
		# $(ANACONDA_HOME)/include/python2.7 \
		# $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include

# Uncomment to use Python 3 (default is Python 2)
PYTHON_LIBRARIES := boost_python-py36 python3.6m
PYTHON_INCLUDE := /usr/include/python3.6m \
                 /usr/lib/python3.6/dist-packages/numpy/core/include

# We need to be able to find libpythonX.X.so or .dylib.
PYTHON_LIB := /usr/lib
# PYTHON_LIB := $(ANACONDA_HOME)/lib

# Homebrew installs numpy in a non standard path (keg only)
# PYTHON_INCLUDE += $(dir $(shell python -c 'import numpy.core; print(numpy.core.__file__)'))/include
# PYTHON_LIB += $(shell brew --prefix numpy)/lib

# Uncomment to support layers written in Python (will link against Python libs)
WITH_PYTHON_LAYER := 1

# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial /usr/include/opencv4
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/aarch64-linux-gnu /usr/lib/aarch64-linux-gnu/hdf5/serial

# If Homebrew is installed at a non standard location (for example your home directory) and you use it for general dependencies
# INCLUDE_DIRS += $(shell brew --prefix)/include
# LIBRARY_DIRS += $(shell brew --prefix)/lib

# NCCL acceleration switch (uncomment to build with NCCL)
# https://github.com/NVIDIA/nccl (last tested version: v1.2.3-1+cuda8.0)
# USE_NCCL := 1

# Uncomment to use `pkg-config` to specify OpenCV library paths.
# (Usually not necessary -- OpenCV libraries are normally installed in one of the above $LIBRARY_DIRS.)
# USE_PKG_CONFIG := 1

# N.B. both build and distribute dirs are cleared on `make clean`
BUILD_DIR := build
DISTRIBUTE_DIR := distribute

# Uncomment for debugging. Does not work on OSX due to https://github.com/BVLC/caffe/issues/171
# DEBUG := 1

# The ID of the GPU that 'make runtest' will use to run unit tests.
TEST_GPUID := 0

# enable pretty build (comment to see full commands)
Q ?= @
```

Makefile

```makefile
PROJECT := caffe

CONFIG_FILE := Makefile.config
# Explicitly check for the config file, otherwise make -k will proceed anyway.
ifeq ($(wildcard $(CONFIG_FILE)),)
$(error $(CONFIG_FILE) not found. See $(CONFIG_FILE).example.)
endif
include $(CONFIG_FILE)

BUILD_DIR_LINK := $(BUILD_DIR)
ifeq ($(RELEASE_BUILD_DIR),)
	RELEASE_BUILD_DIR := .$(BUILD_DIR)_release
endif
ifeq ($(DEBUG_BUILD_DIR),)
	DEBUG_BUILD_DIR := .$(BUILD_DIR)_debug
endif

DEBUG ?= 0
ifeq ($(DEBUG), 1)
	BUILD_DIR := $(DEBUG_BUILD_DIR)
	OTHER_BUILD_DIR := $(RELEASE_BUILD_DIR)
else
	BUILD_DIR := $(RELEASE_BUILD_DIR)
	OTHER_BUILD_DIR := $(DEBUG_BUILD_DIR)
endif

# All of the directories containing code.
SRC_DIRS := $(shell find * -type d -exec bash -c "find {} -maxdepth 1 \
	\( -name '*.cpp' -o -name '*.proto' \) | grep -q ." \; -print)

# The target shared library name
LIBRARY_NAME := $(PROJECT)
LIB_BUILD_DIR := $(BUILD_DIR)/lib
STATIC_NAME := $(LIB_BUILD_DIR)/lib$(LIBRARY_NAME).a
DYNAMIC_VERSION_MAJOR 		:= 1
DYNAMIC_VERSION_MINOR 		:= 0
DYNAMIC_VERSION_REVISION 	:= 0
DYNAMIC_NAME_SHORT := lib$(LIBRARY_NAME).so
#DYNAMIC_SONAME_SHORT := $(DYNAMIC_NAME_SHORT).$(DYNAMIC_VERSION_MAJOR)
DYNAMIC_VERSIONED_NAME_SHORT := $(DYNAMIC_NAME_SHORT).$(DYNAMIC_VERSION_MAJOR).$(DYNAMIC_VERSION_MINOR).$(DYNAMIC_VERSION_REVISION)
DYNAMIC_NAME := $(LIB_BUILD_DIR)/$(DYNAMIC_VERSIONED_NAME_SHORT)
COMMON_FLAGS += -DCAFFE_VERSION=$(DYNAMIC_VERSION_MAJOR).$(DYNAMIC_VERSION_MINOR).$(DYNAMIC_VERSION_REVISION)

##############################
# Get all source files
##############################
# CXX_SRCS are the source files excluding the test ones.
CXX_SRCS := $(shell find src/$(PROJECT) ! -name "test_*.cpp" -name "*.cpp")
# CU_SRCS are the cuda source files
CU_SRCS := $(shell find src/$(PROJECT) ! -name "test_*.cu" -name "*.cu")
# TEST_SRCS are the test source files
TEST_MAIN_SRC := src/$(PROJECT)/test/test_caffe_main.cpp
TEST_SRCS := $(shell find src/$(PROJECT) -name "test_*.cpp")
TEST_SRCS := $(filter-out $(TEST_MAIN_SRC), $(TEST_SRCS))
TEST_CU_SRCS := $(shell find src/$(PROJECT) -name "test_*.cu")
GTEST_SRC := src/gtest/gtest-all.cpp
# TOOL_SRCS are the source files for the tool binaries
TOOL_SRCS := $(shell find tools -name "*.cpp")
# EXAMPLE_SRCS are the source files for the example binaries
EXAMPLE_SRCS := $(shell find examples -name "*.cpp")
# BUILD_INCLUDE_DIR contains any generated header files we want to include.
BUILD_INCLUDE_DIR := $(BUILD_DIR)/src
# PROTO_SRCS are the protocol buffer definitions
PROTO_SRC_DIR := src/$(PROJECT)/proto
PROTO_SRCS := $(wildcard $(PROTO_SRC_DIR)/*.proto)
# PROTO_BUILD_DIR will contain the .cc and obj files generated from
# PROTO_SRCS; PROTO_BUILD_INCLUDE_DIR will contain the .h header files
PROTO_BUILD_DIR := $(BUILD_DIR)/$(PROTO_SRC_DIR)
PROTO_BUILD_INCLUDE_DIR := $(BUILD_INCLUDE_DIR)/$(PROJECT)/proto
# NONGEN_CXX_SRCS includes all source/header files except those generated
# automatically (e.g., by proto).
NONGEN_CXX_SRCS := $(shell find \
	src/$(PROJECT) \
	include/$(PROJECT) \
	python/$(PROJECT) \
	matlab/+$(PROJECT)/private \
	examples \
	tools \
	-name "*.cpp" -or -name "*.hpp" -or -name "*.cu" -or -name "*.cuh")
LINT_SCRIPT := scripts/cpp_lint.py
LINT_OUTPUT_DIR := $(BUILD_DIR)/.lint
LINT_EXT := lint.txt
LINT_OUTPUTS := $(addsuffix .$(LINT_EXT), $(addprefix $(LINT_OUTPUT_DIR)/, $(NONGEN_CXX_SRCS)))
EMPTY_LINT_REPORT := $(BUILD_DIR)/.$(LINT_EXT)
NONEMPTY_LINT_REPORT := $(BUILD_DIR)/$(LINT_EXT)
# PY$(PROJECT)_SRC is the python wrapper for $(PROJECT)
PY$(PROJECT)_SRC := python/$(PROJECT)/_$(PROJECT).cpp
PY$(PROJECT)_SO := python/$(PROJECT)/_$(PROJECT).so
PY$(PROJECT)_HXX := include/$(PROJECT)/layers/python_layer.hpp
# MAT$(PROJECT)_SRC is the mex entrance point of matlab package for $(PROJECT)
MAT$(PROJECT)_SRC := matlab/+$(PROJECT)/private/$(PROJECT)_.cpp
ifneq ($(MATLAB_DIR),)
	MAT_SO_EXT := $(shell $(MATLAB_DIR)/bin/mexext)
endif
MAT$(PROJECT)_SO := matlab/+$(PROJECT)/private/$(PROJECT)_.$(MAT_SO_EXT)

##############################
# Derive generated files
##############################
# The generated files for protocol buffers
PROTO_GEN_HEADER_SRCS := $(addprefix $(PROTO_BUILD_DIR)/, \
		$(notdir ${PROTO_SRCS:.proto=.pb.h}))
PROTO_GEN_HEADER := $(addprefix $(PROTO_BUILD_INCLUDE_DIR)/, \
		$(notdir ${PROTO_SRCS:.proto=.pb.h}))
PROTO_GEN_CC := $(addprefix $(BUILD_DIR)/, ${PROTO_SRCS:.proto=.pb.cc})
PY_PROTO_BUILD_DIR := python/$(PROJECT)/proto
PY_PROTO_INIT := python/$(PROJECT)/proto/__init__.py
PROTO_GEN_PY := $(foreach file,${PROTO_SRCS:.proto=_pb2.py}, \
		$(PY_PROTO_BUILD_DIR)/$(notdir $(file)))
# The objects corresponding to the source files
# These objects will be linked into the final shared library, so we
# exclude the tool, example, and test objects.
CXX_OBJS := $(addprefix $(BUILD_DIR)/, ${CXX_SRCS:.cpp=.o})
CU_OBJS := $(addprefix $(BUILD_DIR)/cuda/, ${CU_SRCS:.cu=.o})
PROTO_OBJS := ${PROTO_GEN_CC:.cc=.o}
OBJS := $(PROTO_OBJS) $(CXX_OBJS) $(CU_OBJS)
# tool, example, and test objects
TOOL_OBJS := $(addprefix $(BUILD_DIR)/, ${TOOL_SRCS:.cpp=.o})
TOOL_BUILD_DIR := $(BUILD_DIR)/tools
TEST_CXX_BUILD_DIR := $(BUILD_DIR)/src/$(PROJECT)/test
TEST_CU_BUILD_DIR := $(BUILD_DIR)/cuda/src/$(PROJECT)/test
TEST_CXX_OBJS := $(addprefix $(BUILD_DIR)/, ${TEST_SRCS:.cpp=.o})
TEST_CU_OBJS := $(addprefix $(BUILD_DIR)/cuda/, ${TEST_CU_SRCS:.cu=.o})
TEST_OBJS := $(TEST_CXX_OBJS) $(TEST_CU_OBJS)
GTEST_OBJ := $(addprefix $(BUILD_DIR)/, ${GTEST_SRC:.cpp=.o})
EXAMPLE_OBJS := $(addprefix $(BUILD_DIR)/, ${EXAMPLE_SRCS:.cpp=.o})
# Output files for automatic dependency generation
DEPS := ${CXX_OBJS:.o=.d} ${CU_OBJS:.o=.d} ${TEST_CXX_OBJS:.o=.d} \
	${TEST_CU_OBJS:.o=.d} $(BUILD_DIR)/${MAT$(PROJECT)_SO:.$(MAT_SO_EXT)=.d}
# tool, example, and test bins
TOOL_BINS := ${TOOL_OBJS:.o=.bin}
EXAMPLE_BINS := ${EXAMPLE_OBJS:.o=.bin}
# symlinks to tool bins without the ".bin" extension
TOOL_BIN_LINKS := ${TOOL_BINS:.bin=}
# Put the test binaries in build/test for convenience.
TEST_BIN_DIR := $(BUILD_DIR)/test
TEST_CU_BINS := $(addsuffix .testbin,$(addprefix $(TEST_BIN_DIR)/, \
		$(foreach obj,$(TEST_CU_OBJS),$(basename $(notdir $(obj))))))
TEST_CXX_BINS := $(addsuffix .testbin,$(addprefix $(TEST_BIN_DIR)/, \
		$(foreach obj,$(TEST_CXX_OBJS),$(basename $(notdir $(obj))))))
TEST_BINS := $(TEST_CXX_BINS) $(TEST_CU_BINS)
# TEST_ALL_BIN is the test binary that links caffe dynamically.
TEST_ALL_BIN := $(TEST_BIN_DIR)/test_all.testbin

##############################
# Derive compiler warning dump locations
##############################
WARNS_EXT := warnings.txt
CXX_WARNS := $(addprefix $(BUILD_DIR)/, ${CXX_SRCS:.cpp=.o.$(WARNS_EXT)})
CU_WARNS := $(addprefix $(BUILD_DIR)/cuda/, ${CU_SRCS:.cu=.o.$(WARNS_EXT)})
TOOL_WARNS := $(addprefix $(BUILD_DIR)/, ${TOOL_SRCS:.cpp=.o.$(WARNS_EXT)})
EXAMPLE_WARNS := $(addprefix $(BUILD_DIR)/, ${EXAMPLE_SRCS:.cpp=.o.$(WARNS_EXT)})
TEST_WARNS := $(addprefix $(BUILD_DIR)/, ${TEST_SRCS:.cpp=.o.$(WARNS_EXT)})
TEST_CU_WARNS := $(addprefix $(BUILD_DIR)/cuda/, ${TEST_CU_SRCS:.cu=.o.$(WARNS_EXT)})
ALL_CXX_WARNS := $(CXX_WARNS) $(TOOL_WARNS) $(EXAMPLE_WARNS) $(TEST_WARNS)
ALL_CU_WARNS := $(CU_WARNS) $(TEST_CU_WARNS)
ALL_WARNS := $(ALL_CXX_WARNS) $(ALL_CU_WARNS)

EMPTY_WARN_REPORT := $(BUILD_DIR)/.$(WARNS_EXT)
NONEMPTY_WARN_REPORT := $(BUILD_DIR)/$(WARNS_EXT)

##############################
# Derive include and lib directories
##############################
CUDA_INCLUDE_DIR := $(CUDA_DIR)/include

CUDA_LIB_DIR :=
# add <cuda>/lib64 only if it exists
ifneq ("$(wildcard $(CUDA_DIR)/lib64)","")
	CUDA_LIB_DIR += $(CUDA_DIR)/lib64
endif
CUDA_LIB_DIR += $(CUDA_DIR)/lib

INCLUDE_DIRS += $(BUILD_INCLUDE_DIR) ./src ./include
ifneq ($(CPU_ONLY), 1)
	INCLUDE_DIRS += $(CUDA_INCLUDE_DIR)
	LIBRARY_DIRS += $(CUDA_LIB_DIR)
	LIBRARIES := cudart cublas curand
endif

LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial

# handle IO dependencies
USE_LEVELDB ?= 1
USE_LMDB ?= 1
# This code is taken from https://github.com/sh1r0/caffe-android-lib
USE_HDF5 ?= 1
USE_OPENCV ?= 1

ifeq ($(USE_LEVELDB), 1)
	LIBRARIES += leveldb snappy
endif
ifeq ($(USE_LMDB), 1)
	LIBRARIES += lmdb
endif
# This code is taken from https://github.com/sh1r0/caffe-android-lib
ifeq ($(USE_HDF5), 1)
	LIBRARIES += hdf5_hl hdf5
endif
ifeq ($(USE_OPENCV), 1)
	LIBRARIES += opencv_core opencv_highgui opencv_imgproc

	ifeq ($(OPENCV_VERSION), 4)
		LIBRARIES += opencv_imgcodecs
	endif

endif
PYTHON_LIBRARIES ?= boost_python python2.7
WARNINGS := -Wall -Wno-sign-compare

##############################
# Set build directories
##############################

DISTRIBUTE_DIR ?= distribute
DISTRIBUTE_SUBDIRS := $(DISTRIBUTE_DIR)/bin $(DISTRIBUTE_DIR)/lib
DIST_ALIASES := dist
ifneq ($(strip $(DISTRIBUTE_DIR)),distribute)
		DIST_ALIASES += distribute
endif

ALL_BUILD_DIRS := $(sort $(BUILD_DIR) $(addprefix $(BUILD_DIR)/, $(SRC_DIRS)) \
	$(addprefix $(BUILD_DIR)/cuda/, $(SRC_DIRS)) \
	$(LIB_BUILD_DIR) $(TEST_BIN_DIR) $(PY_PROTO_BUILD_DIR) $(LINT_OUTPUT_DIR) \
	$(DISTRIBUTE_SUBDIRS) $(PROTO_BUILD_INCLUDE_DIR))

##############################
# Set directory for Doxygen-generated documentation
##############################
DOXYGEN_CONFIG_FILE ?= ./.Doxyfile
# should be the same as OUTPUT_DIRECTORY in the .Doxyfile
DOXYGEN_OUTPUT_DIR ?= ./doxygen
DOXYGEN_COMMAND ?= doxygen
# All the files that might have Doxygen documentation.
DOXYGEN_SOURCES := $(shell find \
	src/$(PROJECT) \
	include/$(PROJECT) \
	python/ \
	matlab/ \
	examples \
	tools \
	-name "*.cpp" -or -name "*.hpp" -or -name "*.cu" -or -name "*.cuh" -or \
        -name "*.py" -or -name "*.m")
DOXYGEN_SOURCES += $(DOXYGEN_CONFIG_FILE)


##############################
# Configure build
##############################

# Determine platform
UNAME := $(shell uname -s)
ifeq ($(UNAME), Linux)
	LINUX := 1
else ifeq ($(UNAME), Darwin)
	OSX := 1
	OSX_MAJOR_VERSION := $(shell sw_vers -productVersion | cut -f 1 -d .)
	OSX_MINOR_VERSION := $(shell sw_vers -productVersion | cut -f 2 -d .)
endif

# Linux
ifeq ($(LINUX), 1)
	CXX ?= /usr/bin/g++
	GCCVERSION := $(shell $(CXX) -dumpversion | cut -f1,2 -d.)
	# older versions of gcc are too dumb to build boost with -Wuninitalized
	ifeq ($(shell echo | awk '{exit $(GCCVERSION) < 4.6;}'), 1)
		WARNINGS += -Wno-uninitialized
	endif
	# boost::thread is reasonably called boost_thread (compare OS X)
	# We will also explicitly add stdc++ to the link target.
	LIBRARIES += boost_thread stdc++
	VERSIONFLAGS += -Wl,-soname,$(DYNAMIC_VERSIONED_NAME_SHORT) -Wl,-rpath,$(ORIGIN)/../lib
endif

# OS X:
# clang++ instead of g++
# libstdc++ for NVCC compatibility on OS X >= 10.9 with CUDA < 7.0
ifeq ($(OSX), 1)
	CXX := /usr/bin/clang++
	ifneq ($(CPU_ONLY), 1)
		CUDA_VERSION := $(shell $(CUDA_DIR)/bin/nvcc -V | grep -o 'release [0-9.]*' | tr -d '[a-z ]')
		ifeq ($(shell echo | awk '{exit $(CUDA_VERSION) < 7.0;}'), 1)
			CXXFLAGS += -stdlib=libstdc++
			LINKFLAGS += -stdlib=libstdc++
		endif
		# clang throws this warning for cuda headers
		WARNINGS += -Wno-unneeded-internal-declaration
		# 10.11 strips DYLD_* env vars so link CUDA (rpath is available on 10.5+)
		OSX_10_OR_LATER   := $(shell [ $(OSX_MAJOR_VERSION) -ge 10 ] && echo true)
		OSX_10_5_OR_LATER := $(shell [ $(OSX_MINOR_VERSION) -ge 5 ] && echo true)
		ifeq ($(OSX_10_OR_LATER),true)
			ifeq ($(OSX_10_5_OR_LATER),true)
				LDFLAGS += -Wl,-rpath,$(CUDA_LIB_DIR)
			endif
		endif
	endif
	# gtest needs to use its own tuple to not conflict with clang
	COMMON_FLAGS += -DGTEST_USE_OWN_TR1_TUPLE=1
	# boost::thread is called boost_thread-mt to mark multithreading on OS X
	LIBRARIES += boost_thread-mt
	# we need to explicitly ask for the rpath to be obeyed
	ORIGIN := @loader_path
	VERSIONFLAGS += -Wl,-install_name,@rpath/$(DYNAMIC_VERSIONED_NAME_SHORT) -Wl,-rpath,$(ORIGIN)/../../build/lib
else
	ORIGIN := \$$ORIGIN
endif

# Custom compiler
ifdef CUSTOM_CXX
	CXX := $(CUSTOM_CXX)
endif

# Static linking
ifneq (,$(findstring clang++,$(CXX)))
	STATIC_LINK_COMMAND := -Wl,-force_load $(STATIC_NAME)
else ifneq (,$(findstring g++,$(CXX)))
	STATIC_LINK_COMMAND := -Wl,--whole-archive $(STATIC_NAME) -Wl,--no-whole-archive
else
  # The following line must not be indented with a tab, since we are not inside a target
  $(error Cannot static link with the $(CXX) compiler)
endif

# Debugging
ifeq ($(DEBUG), 1)
	COMMON_FLAGS += -DDEBUG -g -O0
	NVCCFLAGS += -G
else
	COMMON_FLAGS += -DNDEBUG -O2
endif

# cuDNN acceleration configuration.
ifeq ($(USE_CUDNN), 1)
	LIBRARIES += cudnn
	COMMON_FLAGS += -DUSE_CUDNN
endif

# NCCL acceleration configuration
ifeq ($(USE_NCCL), 1)
	LIBRARIES += nccl
	COMMON_FLAGS += -DUSE_NCCL
endif

# configure IO libraries
ifeq ($(USE_OPENCV), 1)
	COMMON_FLAGS += -DUSE_OPENCV
endif
ifeq ($(USE_LEVELDB), 1)
	COMMON_FLAGS += -DUSE_LEVELDB
endif
ifeq ($(USE_LMDB), 1)
	COMMON_FLAGS += -DUSE_LMDB
ifeq ($(ALLOW_LMDB_NOLOCK), 1)
	COMMON_FLAGS += -DALLOW_LMDB_NOLOCK
endif
endif
# This code is taken from https://github.com/sh1r0/caffe-android-lib
ifeq ($(USE_HDF5), 1)
	COMMON_FLAGS += -DUSE_HDF5
endif

# CPU-only configuration
ifeq ($(CPU_ONLY), 1)
	OBJS := $(PROTO_OBJS) $(CXX_OBJS)
	TEST_OBJS := $(TEST_CXX_OBJS)
	TEST_BINS := $(TEST_CXX_BINS)
	ALL_WARNS := $(ALL_CXX_WARNS)
	TEST_FILTER := --gtest_filter="-*GPU*"
	COMMON_FLAGS += -DCPU_ONLY
endif

# Python layer support
ifeq ($(WITH_PYTHON_LAYER), 1)
	COMMON_FLAGS += -DWITH_PYTHON_LAYER
	LIBRARIES += $(PYTHON_LIBRARIES)
endif

# BLAS configuration (default = ATLAS)
BLAS ?= atlas
ifeq ($(BLAS), mkl)
	# MKL
	LIBRARIES += mkl_rt
	COMMON_FLAGS += -DUSE_MKL
	MKLROOT ?= /opt/intel/mkl
	BLAS_INCLUDE ?= $(MKLROOT)/include
	BLAS_LIB ?= $(MKLROOT)/lib $(MKLROOT)/lib/intel64
else ifeq ($(BLAS), open)
	# OpenBLAS
	LIBRARIES += openblas
else
	# ATLAS
	ifeq ($(LINUX), 1)
		ifeq ($(BLAS), atlas)
			# Linux simply has cblas and atlas
			LIBRARIES += cblas atlas
		endif
	else ifeq ($(OSX), 1)
		# OS X packages atlas as the vecLib framework
		LIBRARIES += cblas
		# 10.10 has accelerate while 10.9 has veclib
		XCODE_CLT_VER := $(shell pkgutil --pkg-info=com.apple.pkg.CLTools_Executables | grep 'version' | sed 's/[^0-9]*\([0-9]\).*/\1/')
		XCODE_CLT_GEQ_7 := $(shell [ $(XCODE_CLT_VER) -gt 6 ] && echo 1)
		XCODE_CLT_GEQ_6 := $(shell [ $(XCODE_CLT_VER) -gt 5 ] && echo 1)
		ifeq ($(XCODE_CLT_GEQ_7), 1)
			BLAS_INCLUDE ?= /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/$(shell ls /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ | sort | tail -1)/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers
		else ifeq ($(XCODE_CLT_GEQ_6), 1)
			BLAS_INCLUDE ?= /System/Library/Frameworks/Accelerate.framework/Versions/Current/Frameworks/vecLib.framework/Headers/
			LDFLAGS += -framework Accelerate
		else
			BLAS_INCLUDE ?= /System/Library/Frameworks/vecLib.framework/Versions/Current/Headers/
			LDFLAGS += -framework vecLib
		endif
	endif
endif
INCLUDE_DIRS += $(BLAS_INCLUDE)
LIBRARY_DIRS += $(BLAS_LIB)

LIBRARY_DIRS += $(LIB_BUILD_DIR)

# Automatic dependency generation (nvcc is handled separately)
CXXFLAGS += -MMD -MP

# Complete build flags.
COMMON_FLAGS += $(foreach includedir,$(INCLUDE_DIRS),-I$(includedir))
CXXFLAGS += -pthread -fPIC $(COMMON_FLAGS) $(WARNINGS)
NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
# mex may invoke an older gcc that is too liberal with -Wuninitalized
MATLAB_CXXFLAGS := $(CXXFLAGS) -Wno-uninitialized
LINKFLAGS += -pthread -fPIC $(COMMON_FLAGS) $(WARNINGS)

USE_PKG_CONFIG ?= 1
ifeq ($(USE_PKG_CONFIG), 1)
	PKG_CONFIG := $(shell pkg-config opencv4 --libs)
else
	PKG_CONFIG :=
endif
LDFLAGS += $(foreach librarydir,$(LIBRARY_DIRS),-L$(librarydir)) $(PKG_CONFIG) \
		$(foreach library,$(LIBRARIES),-l$(library))
PYTHON_LDFLAGS := $(LDFLAGS) $(foreach library,$(PYTHON_LIBRARIES),-l$(library))

# 'superclean' target recursively* deletes all files ending with an extension
# in $(SUPERCLEAN_EXTS) below.  This may be useful if you've built older
# versions of Caffe that do not place all generated files in a location known
# to the 'clean' target.
#
# 'supercleanlist' will list the files to be deleted by make superclean.
#
# * Recursive with the exception that symbolic links are never followed, per the
# default behavior of 'find'.
SUPERCLEAN_EXTS := .so .a .o .bin .testbin .pb.cc .pb.h _pb2.py .cuo

# Set the sub-targets of the 'everything' target.
EVERYTHING_TARGETS := all py$(PROJECT) test warn lint
# Only build matcaffe as part of "everything" if MATLAB_DIR is specified.
ifneq ($(MATLAB_DIR),)
	EVERYTHING_TARGETS += mat$(PROJECT)
endif

##############################
# Define build targets
##############################
.PHONY: all lib test clean docs linecount lint lintclean tools examples $(DIST_ALIASES) \
	py mat py$(PROJECT) mat$(PROJECT) proto runtest \
	superclean supercleanlist supercleanfiles warn everything

all: lib tools examples

lib: $(STATIC_NAME) $(DYNAMIC_NAME)

everything: $(EVERYTHING_TARGETS)

linecount:
	cloc --read-lang-def=$(PROJECT).cloc \
		src/$(PROJECT) include/$(PROJECT) tools examples \
		python matlab

lint: $(EMPTY_LINT_REPORT)

lintclean:
	@ $(RM) -r $(LINT_OUTPUT_DIR) $(EMPTY_LINT_REPORT) $(NONEMPTY_LINT_REPORT)

docs: $(DOXYGEN_OUTPUT_DIR)
	@ cd ./docs ; ln -sfn ../$(DOXYGEN_OUTPUT_DIR)/html doxygen

$(DOXYGEN_OUTPUT_DIR): $(DOXYGEN_CONFIG_FILE) $(DOXYGEN_SOURCES)
	$(DOXYGEN_COMMAND) $(DOXYGEN_CONFIG_FILE)

$(EMPTY_LINT_REPORT): $(LINT_OUTPUTS) | $(BUILD_DIR)
	@ cat $(LINT_OUTPUTS) > $@
	@ if [ -s "$@" ]; then \
		cat $@; \
		mv $@ $(NONEMPTY_LINT_REPORT); \
		echo "Found one or more lint errors."; \
		exit 1; \
	  fi; \
	  $(RM) $(NONEMPTY_LINT_REPORT); \
	  echo "No lint errors!";

$(LINT_OUTPUTS): $(LINT_OUTPUT_DIR)/%.lint.txt : % $(LINT_SCRIPT) | $(LINT_OUTPUT_DIR)
	@ mkdir -p $(dir $@)
	@ python $(LINT_SCRIPT) $< 2>&1 \
		| grep -v "^Done processing " \
		| grep -v "^Total errors found: 0" \
		> $@ \
		|| true

test: $(TEST_ALL_BIN) $(TEST_ALL_DYNLINK_BIN) $(TEST_BINS)

tools: $(TOOL_BINS) $(TOOL_BIN_LINKS)

examples: $(EXAMPLE_BINS)

py$(PROJECT): py

py: $(PY$(PROJECT)_SO) $(PROTO_GEN_PY)

$(PY$(PROJECT)_SO): $(PY$(PROJECT)_SRC) $(PY$(PROJECT)_HXX) | $(DYNAMIC_NAME)
	@ echo CXX/LD -o $@ $<
	$(Q)$(CXX) -shared -o $@ $(PY$(PROJECT)_SRC) \
		-o $@ $(LINKFLAGS) -l$(LIBRARY_NAME) $(PYTHON_LDFLAGS) \
		-Wl,-rpath,$(ORIGIN)/../../build/lib

mat$(PROJECT): mat

mat: $(MAT$(PROJECT)_SO)

$(MAT$(PROJECT)_SO): $(MAT$(PROJECT)_SRC) $(STATIC_NAME)
	@ if [ -z "$(MATLAB_DIR)" ]; then \
		echo "MATLAB_DIR must be specified in $(CONFIG_FILE)" \
			"to build mat$(PROJECT)."; \
		exit 1; \
	fi
	@ echo MEX $<
	$(Q)$(MATLAB_DIR)/bin/mex $(MAT$(PROJECT)_SRC) \
			CXX="$(CXX)" \
			CXXFLAGS="\$$CXXFLAGS $(MATLAB_CXXFLAGS)" \
			CXXLIBS="\$$CXXLIBS $(STATIC_LINK_COMMAND) $(LDFLAGS)" -output $@
	@ if [ -f "$(PROJECT)_.d" ]; then \
		mv -f $(PROJECT)_.d $(BUILD_DIR)/${MAT$(PROJECT)_SO:.$(MAT_SO_EXT)=.d}; \
	fi

runtest: $(TEST_ALL_BIN)
	$(TOOL_BUILD_DIR)/caffe
	$(TEST_ALL_BIN) $(TEST_GPUID) --gtest_shuffle $(TEST_FILTER)

pytest: py
	cd python; python -m unittest discover -s caffe/test

mattest: mat
	cd matlab; $(MATLAB_DIR)/bin/matlab -nodisplay -r 'caffe.run_tests(), exit()'

warn: $(EMPTY_WARN_REPORT)

$(EMPTY_WARN_REPORT): $(ALL_WARNS) | $(BUILD_DIR)
	@ cat $(ALL_WARNS) > $@
	@ if [ -s "$@" ]; then \
		cat $@; \
		mv $@ $(NONEMPTY_WARN_REPORT); \
		echo "Compiler produced one or more warnings."; \
		exit 1; \
	  fi; \
	  $(RM) $(NONEMPTY_WARN_REPORT); \
	  echo "No compiler warnings!";

$(ALL_WARNS): %.o.$(WARNS_EXT) : %.o

$(BUILD_DIR_LINK): $(BUILD_DIR)/.linked

# Create a target ".linked" in this BUILD_DIR to tell Make that the "build" link
# is currently correct, then delete the one in the OTHER_BUILD_DIR in case it
# exists and $(DEBUG) is toggled later.
$(BUILD_DIR)/.linked:
	@ mkdir -p $(BUILD_DIR)
	@ $(RM) $(OTHER_BUILD_DIR)/.linked
	@ $(RM) -r $(BUILD_DIR_LINK)
	@ ln -s $(BUILD_DIR) $(BUILD_DIR_LINK)
	@ touch $@

$(ALL_BUILD_DIRS): | $(BUILD_DIR_LINK)
	@ mkdir -p $@

$(DYNAMIC_NAME): $(OBJS) | $(LIB_BUILD_DIR)
	@ echo LD -o $@
	$(Q)$(CXX) -shared -o $@ $(OBJS) $(VERSIONFLAGS) $(LINKFLAGS) $(LDFLAGS)
	@ cd $(BUILD_DIR)/lib; rm -f $(DYNAMIC_NAME_SHORT);   ln -s $(DYNAMIC_VERSIONED_NAME_SHORT) $(DYNAMIC_NAME_SHORT)

$(STATIC_NAME): $(OBJS) | $(LIB_BUILD_DIR)
	@ echo AR -o $@
	$(Q)ar rcs $@ $(OBJS)

$(BUILD_DIR)/%.o: %.cpp $(PROTO_GEN_HEADER) | $(ALL_BUILD_DIRS)
	@ echo CXX $<
	$(Q)$(CXX) $< $(CXXFLAGS) -c -o $@ 2> $@.$(WARNS_EXT) \
		|| (cat $@.$(WARNS_EXT); exit 1)
	@ cat $@.$(WARNS_EXT)

$(PROTO_BUILD_DIR)/%.pb.o: $(PROTO_BUILD_DIR)/%.pb.cc $(PROTO_GEN_HEADER) \
		| $(PROTO_BUILD_DIR)
	@ echo CXX $<
	$(Q)$(CXX) $< $(CXXFLAGS) -c -o $@ 2> $@.$(WARNS_EXT) \
		|| (cat $@.$(WARNS_EXT); exit 1)
	@ cat $@.$(WARNS_EXT)

$(BUILD_DIR)/cuda/%.o: %.cu | $(ALL_BUILD_DIRS)
	@ echo NVCC $<
	$(Q)$(CUDA_DIR)/bin/nvcc $(NVCCFLAGS) $(CUDA_ARCH) -M $< -o ${@:.o=.d} \
		-odir $(@D)
	$(Q)$(CUDA_DIR)/bin/nvcc $(NVCCFLAGS) $(CUDA_ARCH) -c $< -o $@ 2> $@.$(WARNS_EXT) \
		|| (cat $@.$(WARNS_EXT); exit 1)
	@ cat $@.$(WARNS_EXT)

$(TEST_ALL_BIN): $(TEST_MAIN_SRC) $(TEST_OBJS) $(GTEST_OBJ) \
		| $(DYNAMIC_NAME) $(TEST_BIN_DIR)
	@ echo CXX/LD -o $@ $<
	$(Q)$(CXX) $(TEST_MAIN_SRC) $(TEST_OBJS) $(GTEST_OBJ) \
		-o $@ $(LINKFLAGS) $(LDFLAGS) -l$(LIBRARY_NAME) -Wl,-rpath,$(ORIGIN)/../lib

$(TEST_CU_BINS): $(TEST_BIN_DIR)/%.testbin: $(TEST_CU_BUILD_DIR)/%.o \
	$(GTEST_OBJ) | $(DYNAMIC_NAME) $(TEST_BIN_DIR)
	@ echo LD $<
	$(Q)$(CXX) $(TEST_MAIN_SRC) $< $(GTEST_OBJ) \
		-o $@ $(LINKFLAGS) $(LDFLAGS) -l$(LIBRARY_NAME) -Wl,-rpath,$(ORIGIN)/../lib

$(TEST_CXX_BINS): $(TEST_BIN_DIR)/%.testbin: $(TEST_CXX_BUILD_DIR)/%.o \
	$(GTEST_OBJ) | $(DYNAMIC_NAME) $(TEST_BIN_DIR)
	@ echo LD $<
	$(Q)$(CXX) $(TEST_MAIN_SRC) $< $(GTEST_OBJ) \
		-o $@ $(LINKFLAGS) $(LDFLAGS) -l$(LIBRARY_NAME) -Wl,-rpath,$(ORIGIN)/../lib

# Target for extension-less symlinks to tool binaries with extension '*.bin'.
$(TOOL_BUILD_DIR)/%: $(TOOL_BUILD_DIR)/%.bin | $(TOOL_BUILD_DIR)
	@ $(RM) $@
	@ ln -s $(notdir $<) $@

$(TOOL_BINS): %.bin : %.o | $(DYNAMIC_NAME)
	@ echo CXX/LD -o $@
	$(Q)$(CXX) $< -o $@ $(LINKFLAGS) -l$(LIBRARY_NAME) $(LDFLAGS) \
		-Wl,-rpath,$(ORIGIN)/../lib

$(EXAMPLE_BINS): %.bin : %.o | $(DYNAMIC_NAME)
	@ echo CXX/LD -o $@
	$(Q)$(CXX) $< -o $@ $(LINKFLAGS) -l$(LIBRARY_NAME) $(LDFLAGS) \
		-Wl,-rpath,$(ORIGIN)/../../lib

proto: $(PROTO_GEN_CC) $(PROTO_GEN_HEADER)

$(PROTO_BUILD_DIR)/%.pb.cc $(PROTO_BUILD_DIR)/%.pb.h : \
		$(PROTO_SRC_DIR)/%.proto | $(PROTO_BUILD_DIR)
	@ echo PROTOC $<
	$(Q)protoc --proto_path=$(PROTO_SRC_DIR) --cpp_out=$(PROTO_BUILD_DIR) $<

$(PY_PROTO_BUILD_DIR)/%_pb2.py : $(PROTO_SRC_DIR)/%.proto \
		$(PY_PROTO_INIT) | $(PY_PROTO_BUILD_DIR)
	@ echo PROTOC \(python\) $<
	$(Q)protoc --proto_path=src --python_out=python $<

$(PY_PROTO_INIT): | $(PY_PROTO_BUILD_DIR)
	touch $(PY_PROTO_INIT)

clean:
	@- $(RM) -rf $(ALL_BUILD_DIRS)
	@- $(RM) -rf $(OTHER_BUILD_DIR)
	@- $(RM) -rf $(BUILD_DIR_LINK)
	@- $(RM) -rf $(DISTRIBUTE_DIR)
	@- $(RM) $(PY$(PROJECT)_SO)
	@- $(RM) $(MAT$(PROJECT)_SO)

supercleanfiles:
	$(eval SUPERCLEAN_FILES := $(strip \
			$(foreach ext,$(SUPERCLEAN_EXTS), $(shell find . -name '*$(ext)' \
			-not -path './data/*'))))

supercleanlist: supercleanfiles
	@ \
	if [ -z "$(SUPERCLEAN_FILES)" ]; then \
		echo "No generated files found."; \
	else \
		echo $(SUPERCLEAN_FILES) | tr ' ' '\n'; \
	fi

superclean: clean supercleanfiles
	@ \
	if [ -z "$(SUPERCLEAN_FILES)" ]; then \
		echo "No generated files found."; \
	else \
		echo "Deleting the following generated files:"; \
		echo $(SUPERCLEAN_FILES) | tr ' ' '\n'; \
		$(RM) $(SUPERCLEAN_FILES); \
	fi

$(DIST_ALIASES): $(DISTRIBUTE_DIR)

$(DISTRIBUTE_DIR): all py | $(DISTRIBUTE_SUBDIRS)
	# add proto
	cp -r src/caffe/proto $(DISTRIBUTE_DIR)/
	# add include
	cp -r include $(DISTRIBUTE_DIR)/
	mkdir -p $(DISTRIBUTE_DIR)/include/caffe/proto
	cp $(PROTO_GEN_HEADER_SRCS) $(DISTRIBUTE_DIR)/include/caffe/proto
	# add tool and example binaries
	cp $(TOOL_BINS) $(DISTRIBUTE_DIR)/bin
	cp $(EXAMPLE_BINS) $(DISTRIBUTE_DIR)/bin
	# add libraries
	cp $(STATIC_NAME) $(DISTRIBUTE_DIR)/lib
	install -m 644 $(DYNAMIC_NAME) $(DISTRIBUTE_DIR)/lib
	cd $(DISTRIBUTE_DIR)/lib; rm -f $(DYNAMIC_NAME_SHORT);   ln -s $(DYNAMIC_VERSIONED_NAME_SHORT) $(DYNAMIC_NAME_SHORT)
	# add python - it's not the standard way, indeed...
	cp -r python $(DISTRIBUTE_DIR)/

-include $(DEPS)
```

3. make and check for Caffe:

```shell
# j + cpu cores number
make all -j6
make test
make runtest
```

### install pycaffe

1. Compile

```shell
make pycaffe
```
2. Install

The default version of python is 3.6

```shell
sudo cp -r python/caffe/ /usr/local/lib/python3.6/dist-packages/
```

3. Check/Test

Make sure the below command works without any errors

```shell
python3 -c "import caffe"
```

4. Possible errors:

- Check [troubleshooting](#troubleshooting) section, issue 4, 6

### troubleshooting

*Note: Only follow this section if you encounter any error*

1. hdf5.h: No such file or directory

In Makefile.config
- Add `/usr/include/hdf5/serial` to INCLUDE_DIRS (https://github.com/BVLC/caffe/issues/4808)

In Makefile
- Change `hdf5_hl` and `hdf5` in LIBRARIES to `hdf5_serial_hl` and `hdf5_serial` respectively

2. Undefined symbol cblas_*

In Makefile.config
- Change `BLAS := atlas` to `BLAS := open` to use openblas instead

3. ld cannot find lboost_python3

- Create symbolic link from libboost_python-py35.so to libboost_python3 (https://github.com/BVLC/caffe/issues/4843)

4. numpy/arrayobject.h not found

Make sure you get the correct path of Python dist-packages directory. In Ubuntu 17.04 it's in /usr/local/lib. I remember earlier version it's in /usr/lib.
In Makefile.config
- Find PYTHON_INCLUDE, change `/usr/lib/python3.5/dist-packages/numpy/core/include` to `/usr/local/lib/python3.5/dist-packages/numpy/core/include`

5. libcaffe.so undefined reference to caffe::*

- Make sure that you have `make clean` before `make all`.
- Also delete the libcaffe.so* we have copied to /usr/lib
```shell
sudo rm -f /usr/lib/libcaffe.so*
```

6. SyntaxError: invalid syntax in rrule.py (Can't create weekday with n == 0)

Update python-dateutil (https://github.com/dungba88/caffe-python3-install/issues/1)

```shell
pip3 install python-dateutil --upgrade 
```
