# Installing DensePose

Installing DensePose is not an easy thing except [building it in a docker container](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md#docker-image). Here, I provided an installation guide based on the official [DensePose Installation](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md) and an [installation guide](http://linkinpark213.com/2018/11/18/densepose-minesweeping/) provided by [@linkinpark213](https://github.com/linkinpark213).


### Contents
* [Requirements](#requirements)
* [Notes](#notes)
* [Install Dependencies](#install-dependencies)
    * [Python Requirements (via Conda)](#python-requirements-via-conda)
    * [COCO API](#coco-api)
    * [PyTorch & Caffe2](#pytorch--caffe2)
        * [Install from Source Code](#install-from-source-code)
        * [Install from Binaries via Conda](#install-from-binaries-via-conda)
        * [Install protobuf package](#install-protobuf-package)
        * [Caffe2 Installation Test](#caffe2-installation-test)
    * [GCC compiler (optional)](#gcc-compiler-optional)
* [DensePose](#densepose)
    * [Install DensePose](#install-densepose)
    * [Fetch DensePose data](#fetch-densepose-data)
* [Setting-up the COCO dataset](#setting-up-the-coco-dataset)
* [Acknowledgements](#acknowledgements)


---

#### Updates:
##### 2019.5.13
* Upgrade to PyTorch-1.1.0.
* Upgrade `mkl` to the latest version (2019.3-199) since relevant bug has been fixed.
* Modify the installation instructions of `protobuf` package.

#### Requirements:
- NVIDIA GPU, Linux, Python 2/3 (It is highly recommended to install an [Anaconda](https://www.anaconda.com/distribution/#download-section) environment)
- [Caffe2](#pytorch--caffe2), various standard [Python packages](#python-requirements-via-conda)
- [COCO API](#coco-api)
- [cmake 3.8.2](https://cmake.org/files/), [GCC-4.9.2](https://gcc.gnu.org/mirrors.html), [protobuf](https://github.com/protocolbuffers/protobuf).

#### Notes:
- [Detectron operators](https://github.com/pytorch/pytorch/tree/master/modules/detectron) currently do not have CPU implementation; a GPU system is required.
- The latest [facebookresearch/Detectron](https://github.com/facebookresearch/Detectron) has already enabled **Python 3** support, so I updated the detectron module in DensePose to make it compatible in Python 3 and refined a lot of codes in original DensePose repo. If you prefer Python 3, you can build DensePose using code in this [repo](https://github.com/Johnqczhang/DensePose).

#### My System Environment
  * System: CentOS 7.0
  * GPU: NVIDIA Tesla P100
  * CUDA: 9.0 with cuDNN 7.4.2
  * Python: 3.7, based on conda 4.6.14.


## Install Dependencies

### Python Requirements (via Conda)
It is recommended to create a python environment with a speficied python version (e.g., 2.7/3.6/3.7) via Conda by this following command,
```bash
$ conda create -y -n your_env_name python=x.x
```
Note that instructions like `x.x` or `pythonx.x` (below) indicate the specific python version you installed in your created conda environment, which can be 2.7/3.6/3.7 or any other valid python version.

For convenience in the following instructions, we set some environment variables accordingly: 
```bash
export CONDA_ENV_PATH=/path/to/anaconda3/envs/your_env_name
export CONDA_PKGS_PATH=/path/to/anaconda3/envs/your_env_name/lib/pythonx.x/site-packages
```

Then, install the following python packages under the created environment:
- PyTorch: `conda install -y numpy setuptools cffi typing pyyaml=3.13 mkl=2019.1 mkl-include=2019.1`
- COCOAPI: `conda install -y cython matplotlib`
- Caffe2: `conda install -y pydot future networkx`
- DensePose: `conda install -y opencv mock scipy h5py`
- [SMPL](http://smpl.is.tue.mpg.de/) model: `pip install chumpy`

By default, conda will install the latest version of these packages. However, package `pyyaml` with version 4.x may cause a `ConstructorError` when loading a `.yaml` file according to this [issue](https://github.com/facebookresearch/DensePose/issues/216), current solution is downgrade `pyyaml` to version 3.13 (>= 3.12).


### [COCO API](https://github.com/cocodataset/cocoapi)

```bash
# COCOAPI=/path/to/clone/cocoapi
$ git clone https://github.com/cocodataset/cocoapi.git $COCOAPI
$ cd $COCOAPI/PythonAPI
# Install into global site-packages
$ make install
```
Note that instructions like `# COCOAPI=/path/to/install/cocoapi` indicate that you should pick a path where you'd like to have the software cloned and then set an environment variable (`COCOAPI` in this case) accordingly.


### PyTorch & Caffe2
[Caffe2](https://caffe2.ai/) has been merged and integrated into [PyTorch](https://pytorch.org/), you can install them either from source code or from binaries via Conda.

#### Install from Source Code
See [Install PyTorch from Source](https://github.com/pytorch/pytorch#from-source).

#### Install from Binaries via Conda
Commands to install from binaries via Conda or pip wheels are on [PyTorch website](https://pytorch.org). E.g. (in my case), 
```bash
$ conda install pytorch torchvision cudatoolkit=9.0 -c pytorch
```

#### Install protobuf package
When the installation of pytorch & caffe2 is done, the python API package (named `torch` and `caffe2`) is installed in `$CONDA_PKGS_PATH`, for convenience in the following instructions, set some environment variables:
```bash
export TORCH_PATH=$CONDA_PKGS_PATH/torch
export CAFFE2_INCLUDE_PATH=$TORCH_PATH/include/caffe2
```

Before installing protobuf, you need to check which version (currently, 3.6.1) is specified in the header of `$CAFFE2_INCLUDE_PATH/proto/caffe2.pb.h` like the following code,
```h
// Generated by the protocol buffer compiler.  DO NOT EDIT!
// source: caffe2/proto/caffe2.proto

#ifndef PROTOBUF_INCLUDED_caffe2_2fproto_2fcaffe2_2eproto
#define PROTOBUF_INCLUDED_caffe2_2fproto_2fcaffe2_2eproto

#include <string>

#include <google/protobuf/stubs/common.h>

#if GOOGLE_PROTOBUF_VERSION < 3006001
#error This file was generated by a newer version of protoc which is
#error incompatible with your Protocol Buffer headers.  Please update
#error your headers.
#endif
#if 3006001 < GOOGLE_PROTOBUF_MIN_PROTOC_VERSION
#error This file was generated by an older version of protoc which is
#error incompatible with your Protocol Buffer headers.  Please
#error regenerate this file with a newer version of protoc.
#endif
```

Then, install protobuf via conda by `conda install protobuf=3.6.1` accordingly.

#### Caffe2 Installation Test
~~Adjust your `PYTHONPATH` environment variable to include this location: `$TORCH_PATH/lib` (referred to the [issue: Detectron ops lib not found](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-2-Detectron-ops-lib-not-found)). If this doesn't work when [setting up python modules of densepose](#densepose), try creating a symbol link to this path in the root directory of your densepose: `ln -s $TORCH_PATH/lib /path/to/your/densepose/lib`~~ (This issue has been solved by the official [Detectron](https://github.com/facebookresearch/Detectron/commit/df0e4972de432841fa0bf3a99ef86ac5b99471b7#diff-c9dcb56d00bdc377a5cde4915aba114f)).

Please ensure that your Caffe2 installation and [protobuf installation](#install-protobuf-package) was successful before proceeding by running the following commands and checking their output as directed in the comments.

```bash
# To check if Caffe2 build was successful
$ python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
$ python -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
```

To avoid this [issue](https://github.com/facebookresearch/DensePose/issues/185) in case multiple CUDA libraries have been installed in your system and the symbol link `/usr/local/cuda` doesn't link to the specific version of CUDA that you're using, you need to replace `/usr/local/cuda/lib64/libculibos.a` with `/usr/local/cuda-x.x/lib64/libculibos.a` in `$TORCH_PATH/share/cmake/Caffe2/Caffe2Target.cmake`, otherwise it may trigger a linking error when [installing the custom DensePose operator](#build-the-custom-operators-library). (**This issue only occurs in which the caffe2 was installed from binaries via conda**).


### GCC compiler (optional)
If Caffe2 was installed via Conda, then you need to make sure that you have the feasible GCC compiler in your system before building densepose (referred to this [issue](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-10-Undefined-symbol-ZN6caffe219CPUOperatorRegistryB5cxx11Ev)). 
So far, [GCC-4.9.2](https://ftp.gnu.org/gnu/gcc/gcc-4.9.2/) has been successfully tested to be used to compile custom operators in densepose which depends on Caffe2 libraries. You can also run the following command to see which versions of GCC are supported in the precompiled `libcaffe2.so` library like the following output,
```bash
$ strings -a $TORCH_PATH/lib/libcaffe2.so | grep "GCC: ("
$ # Output from the command above
$ GCC: (GNU) 4.9.2 20150212 (Red Hat 4.9.2-6)`
```

#### Install GCC
Download [GCC-4.9.2](https://ftp.gnu.org/gnu/gcc/gcc-4.9.2/) source code and build it by the following commands,
```bash
$ mkdir /path/to/gcc-4.9.2/build && cd /path/to/gcc-4.9.2/build
$ ../configure --prefix=/path/to/build --enable-checking=release --enable-languages=c,c++ --disable-multilib
$ make
$ make install
$ cd bin/
$ ln -s gcc cc  # create a symbol link 'cc' for 'gcc'
$ cp -r include/c++ $CONDA_ENV_PATH/include
```

After installing gcc-4.9.2 successfully, add `/path/to/gcc-4.9.2/build/bin` and `/path/to/gcc-4.9.2/build/lib64` to your `$PATH` and `$LD_LIBRARY_PATH` environment variables, respectively.


## Densepose

### Install DensePose
Clone the Densepose repository:

```bash
# DENSEPOSE=/path/to/clone/densepose
# official DensePose repo
$ git clone https://github.com/facebookresearch/densepose $DENSEPOSE
# or clone my forked repo with python3 compatibility support
$ git clone https://github.com/Johnqczhang/DensePose $DENSEPOSE
```

Set up Python modules:

```bash
$ cd $DENSEPOSE
$ python setup.py develop
```

Check that Detectron tests pass (e.g. for [`SpatialNarrowAsOp test`](https://github.com/facebookresearch/DensePose/blob/master/detectron/tests/test_spatial_narrow_as_op.py)):

```bash
$ python $DENSEPOSE/detectron/tests/test_spatial_narrow_as_op.py
```

#### Build the custom operators library

1. Edit `$DENSEPOSE/CMakeLists.txt` (Thanks to [@hyousamk's solution](https://github.com/facebookresearch/DensePose/issues/119)), you can download [`CMakeLists.txt`](https://github.com/Johnqczhang/densepose_installation/blob/master/CMakeLists.txt) from this repository into your `$DENSEPOSE`, and replace corresponding paths specified in this file with yours:
    
    ```diff
    diff --git a/CMakeLists.txt b/CMakeLists.txt
    index 488ea86..b59d9bb 100644
    --- a/CMakeLists.txt
    +++ b/CMakeLists.txt
    @@ -1,11 +1,24 @@
    cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

    +# set caffe2 cmake path manually
    +set(Caffe2_DIR "/path/to/anaconda3/envs/env_name/lib/pythonx.x/site-packages/torch/share/cmake/Caffe2")
    +# set cuDNN path
    +set(CUDNN_INCLUDE_DIR "/path/to/your/cudnn/include")
    +set(CUDNN_LIBRARY "/path/to/your/libcudnn/libcudnn.so")
    +include_directories("/path/to/anaconda3/envs/env_name/include")
    +# add static protobuf library
    +add_library(libprotobuf STATIC IMPORTED)
    +set(PROTOBUF_LIB "/path/to/anaconda3/envs/env_name/lib/libprotobuf.a")
    +set_property(TARGET libprotobuf PROPERTY IMPORTED_LOCATION "${PROTOBUF_LIB}")
    +
    # Find the Caffe2 package.
    # Caffe2 exports the required targets, so find_package should work for
    # the standard Caffe2 installation. If you encounter problems with finding
    # the Caffe2 package, make sure you have run `make install` when installing
    # Caffe2 (`make install` populates your share/cmake/Caffe2).
    find_package(Caffe2 REQUIRED)
    +include_directories(${CAFFE2_INCLUDE_DIRS})

    if (${CAFFE2_VERSION} VERSION_LESS 0.8.2)
    # Pre-0.8.2 caffe2 does not have proper interface libraries set up, so we
    @@ -34,19 +47,19 @@ add_library(
        caffe2_detectron_custom_ops SHARED
        ${CUSTOM_OPS_CPU_SRCS})

    -target_link_libraries(caffe2_detectron_custom_ops caffe2_library)
    +target_link_libraries(caffe2_detectron_custom_ops caffe2_library libprotobuf)
    install(TARGETS caffe2_detectron_custom_ops DESTINATION lib)

    # Install custom GPU ops lib, if gpu is present.
    if (CAFFE2_USE_CUDA OR CAFFE2_FOUND_CUDA)
    # Additional -I prefix is required for CMake versions before commit (< 3.7):
    # https://github.com/Kitware/CMake/commit/7ded655f7ba82ea72a82d0555449f2df5ef38594
    -  list(APPEND CUDA_INCLUDE_DIRS -I${CAFFE2_INCLUDE_DIRS})
    +  # list(APPEND CUDA_INCLUDE_DIRS -I${CAFFE2_INCLUDE_DIRS})
    CUDA_ADD_LIBRARY(
        caffe2_detectron_custom_ops_gpu SHARED
        ${CUSTOM_OPS_CPU_SRCS}
        ${CUSTOM_OPS_GPU_SRCS})

    -  target_link_libraries(caffe2_detectron_custom_ops_gpu caffe2_gpu_library)
    +  target_link_libraries(caffe2_detectron_custom_ops_gpu caffe2_gpu_library libprotobuf)
    install(TARGETS caffe2_detectron_custom_ops_gpu DESTINATION lib)
    endif()
    ```

2. Download the source code of [PyTorch](https://github.com/pytorch/pytorch), then copy the following folders into `$CAFFE2_INCLUDE_PATH/utils/` (referred to this [issue](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-8-fatal-error-caffe2-utils-threadpool-ThreadPool-h-No-such-file-or-directory) and this [issue](https://github.com/Johnqczhang/densepose_installation/issues/3) which may be related to recent updates of PyTorch):
    ```bash
    $ cp -r $PYTORCH/caffe2/utils/threadpool $CAFFE2_INCLUDE_PATH/utils/
    $ cp -r $PYTORCH/caffe2/utils/math $CAFFE2_INCLUDE_PATH/utils/
    ```

3. Compile the custom operator:
    ```bash
    $ cd $DENSEPOSE/build
    $ cmake .. && make
    ```

4. Check that the custom operator tests pass:
    ```bash
    $ python $DENSEPOSE/detectron/tests/test_zero_even_op.py
    ```
    If you installed the custom operator successfully without any error, you will see the following results in your console:
    ```bash
    $ python detectron/tests/test_zero_even_op.py
    [E init_intrinsics_check.cc:43] CPU feature avx is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
    [E init_intrinsics_check.cc:43] CPU feature avx2 is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
    [E init_intrinsics_check.cc:43] CPU feature fma is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
    ............
    ----------------------------------------------------------------------
    Ran 12 tests in 3.155s

    OK
    ```

### Fetch DensePose data
Please read [Fetch DensePose data](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md#fetch-densepose-data) in the official installation documentation.

## Setting-up the COCO dataset
Please read [Setting-up the COCO dataset](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md#setting-up-the-coco-dataset) in the official installation documentation.

---

Congratulations! Now you've installed DensePose successfully! :tada:

Happy Hunting DensePose!

## Acknowledgements
- [DensePose Github](https://github.com/facebookresearch/DensePose)
- [[MineSweeping] The Long Struggle of DensePose Installation](http://linkinpark213.com/2018/11/18/densepose-minesweeping/), a very helpful post about the DensePose installation provided by [@linkinpark213](https://github.com/linkinpark213).
- Special thanks to [@tete1030](https://github.com/tete1030) for his great help in debugging the installation of custom operator with gcc compiler.
