# Installing DensePose

Installing DensePose is not an easy thing except building it from the provided [Dockerfile](https://github.com/facebookresearch/DensePose/blob/master/docker/Dockerfile). Here, I provided an installation guide based on the official [DensePose Installation](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md) and an [installation guide](http://linkinpark213.com/2018/11/18/densepose-minesweeping/) provided by [@linkinpark213](https://github.com/linkinpark213).


### Contents
* [Requirements](#requirements)
* [Notes](#notes)
* [Install Dependencies](#install-dependencies)
    * [Python Requirements](#python-requirements)
    * [GCC compiler](#gcc-compiler)
    * [Google Protocol Buffers (protobuf)](#google-protocol-buffers-protobuf)
    * [COCO API](#coco-api)
* [Caffe2](#caffe2)
    * [Install Caffe2 from Source Code](#install-caffe2-from-source-code)
    * [Install Caffe2 from Binaries via Conda](#install-caffe2-from-binaries-via-conda)
    * [Caffe2 Installation Test](#caffe2-installation-test)
* [DensePose](#densepose)
    * [Install DensePose](#install-densepose)
    * [Fetch DensePose data](#fetch-densepose-data)
* [Setting-up the COCO dataset](#setting-up-the-coco-dataset)
* [Acknowledgements](#acknowledgements)


---

#### Requirements:
- NVIDIA GPU, Linux, Python 2/3 (It is highly recommended to install an [Anaconda](https://www.continuum.io/downloads) environment)
- Caffe2, various standard Python packages, and the COCO API; Instructions for installing these dependencies are found below.
- [cmake 3.8.2](https://cmake.org/files/), [GCC-4.9.2](https://gcc.gnu.org/mirrors.html), [protobuf-3.5.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.5.0). These versions of dependencies are exactly specified in caffe2 installed from binaries via conda. If you choose to [install caffe2 from source code](#install-caffe2-from-source-code), then you don't need to install these packages. If you choose to [install caffe2 from binaries via conda](#install-caffe2-from-binaries-via-conda), then follow instructions for installing these dependencies which are found below.

#### Notes:
- [Detectron operators](https://github.com/pytorch/pytorch/tree/master/modules/detectron) currently do not have CPU implementation; a GPU system is required.
- The latest [facebookresearch/Detectron](https://github.com/facebookresearch/Detectron) has already enabled Python 3 support, so I updated the detectron module in DensePose to make it compatible in Python 3 and also refined a lot of codes in original DensePose repo. If you prefer Python 3, you can build DensePose using code in this [repo](https://github.com/Johnqczhang/DensePose).


#### My System Environment
  * System: CentOS 7.0
  * GPU: NVIDIA Tesla P100
  * CUDA: 9.0 with cuDNN 7.4.2
  * Python: 2.7.15 / 3.7, based on conda 4.6.11. (I installed [Anaconda3](https://www.anaconda.com/distribution/#download-section) and created an environment with a specified python version by `conda create -n env_name python=x.x` (`x.x` can be 2.7/3.5/3.6/3.7). For convenience in the following instructions, set `CONDA_ENV_PATH=/path/to/anaconda3/envs/env_name`.

## Install Dependencies

### Python Requirements
- PyTorch: `conda install -y numpy setuptools cffi typing pyyaml=3.13 mkl=2019.1 mkl-include=2019.1`
- COCOAPI: `conda install -y cython matplotlib`
- Caffe2: `conda install -y pydot future networkx`
- Caffe2 ([from source](#install-caffe2-from-source-code)): `conda install protobuf=3.6.1`
- DensePose: `conda install -y opencv mock scipy h5py`
- [SMPL](http://smpl.is.tue.mpg.de/) model: `pip install -y chumpy`

By default, conda will install the latest version of packages. However, package `pyyaml` with version 4.x may cause a `ConstructorError` when loading a `.yaml` file according to this [issue](https://github.com/facebookresearch/DensePose/issues/216), current solution is downgrade `pyyaml` to version 3.13 (>= 3.12).

The latest `mkl` packages (version: 2019.3) may has some bugs which caused errors when building PyTorch from source in my environment. So I downgrade it to a previous version.


### GCC compiler    
So far, [GCC-4.9.2](https://ftp.gnu.org/gnu/gcc/gcc-4.9.2/) was used to compile caffe2 library in the precompiled conda package (referred to this [issue](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-10-Undefined-symbol-ZN6caffe219CPUOperatorRegistryB5cxx11Ev)). You can firstly [install caffe2](#caffe2) and run the following command to see which version of GCC was used in the precompiled `libcaffe2.so` library like the following output,
```bash
$ strings -a $TORCH_PATH/lib/libcaffe2.so | grep "GCC: ("
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

### Google Protocol Buffers (protobuf)
So far, [profobuf-3.5.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.5.0) was specified in caffe2 installed from binaries via conda. It is highly recommended to firstly [install caffe2](#caffe2) and check which version of protobuf was specified in the header of `$TORCH_PATH/lib/include/caffe2/proto/caffe2.pb.h` like the following example, then install the corresponding verison of protobuf like the way below:

```h
// Generated by the protocol buffer compiler.  DO NOT EDIT!
// source: caffe2/proto/caffe2.proto

#ifndef PROTOBUF_caffe2_2fproto_2fcaffe2_2eproto__INCLUDED
#define PROTOBUF_caffe2_2fproto_2fcaffe2_2eproto__INCLUDED

#include <string>

#include <google/protobuf/stubs/common.h>

#if GOOGLE_PROTOBUF_VERSION < 3005000
#error This file was generated by a newer version of protoc which is
#error incompatible with your Protocol Buffer headers.  Please update
#error your headers.
#endif
#if 3005000 < GOOGLE_PROTOBUF_MIN_PROTOC_VERSION
#error This file was generated by an older version of protoc which is
#error incompatible with your Protocol Buffer headers.  Please
#error regenerate this file with a newer version of protoc.
#endif
```

#### Install protobuf C/C++ API
```bash
$ mkdir /path/to/protobuf-3.5.0/build && cd /path/to/protobuf-3.5.0/build
$ ../configure --prefix=/path/to/build CXXFLAGS="-fPIC"
$ make
$ make install
```

Like what we did in the GCC installation, after the installation is done, add `/path/to/protobuf-3.5.0/build/bin` and `/path/to/protobuf-3.5.0/build/lib` to your `$PATH` and `$LD_LIBRARY_PATH` environment variables. You can also copy these built files to the corresponding directories of your conda environment which will only take effect within the created environment (I did in this way),
```bash
$ cd /path/to/protobuf-3.5.0/build
$ cp bin/protoc $CONDA_ENV_PATH/bin
$ cp -r include/google $CONDA_ENV_PATH/include
$ cp lib/libproto* $CONDA_ENV_PATH/lib
```

#### Install protobuf Python API
Before this installation, make sure that you're in the correct python environment (i.e., `env_name`).
```bash
$ cd /path/to/protobuf-3.5.0/python
$ python setup.py build
$ python setup.py install
```

### [COCO API](https://github.com/cocodataset/cocoapi)

```bash
# COCOAPI=/path/to/clone/cocoapi
$ git clone https://github.com/cocodataset/cocoapi.git $COCOAPI
$ cd $COCOAPI/PythonAPI
# Install into global site-packages
$ make install
```
Note that instructions like `# COCOAPI=/path/to/install/cocoapi` indicate that you should pick a path where you'd like to have the software cloned and then set an environment variable (`COCOAPI` in this case) accordingly.


## Caffe2
[Caffe2](https://caffe2.ai/) has been merged and integrated into [PyTorch](https://pytorch.org/), you can install it either from binaries via Conda or from source code.
### Install Caffe2 from Source Code
See [Install PyTorch from Source](https://github.com/pytorch/pytorch#from-source).

If you are installing from source, by default it will also build a protobuf (`protobuf-3.6.1` in my case), so you don't need to [install protobuf-3.5.0](#google-protocol-buffers-protobuf) from source code as specified above but just simply run this command `conda install protobuf=3.6.1`.

### Install Caffe2 from Binaries via Conda
Commands to install from binaries via Conda or pip wheels are on PyTorch website: https://pytorch.org

### Caffe2 Installation Test
If you install pytorch and caffe2 successfully, you will see there are two folders named `torch` and `caffe2` in your conda packages directory (`$CONDA_ENV_PATH/lib/pythonx.x/site-packages/`)
For convenience, set `TORCH_PATH=$CONDA_ENV_PATH/lib/pythonx.x/site-packages/torch`.

~~Adjust your `PYTHONPATH` environment variable to include this location: `$TORCH_PATH/lib` (referred to the [issue: Detectron ops lib not found](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-2-Detectron-ops-lib-not-found)). If this doesn't work when [setting up python modules of densepose](#densepose), try creating a symbol link to this path in the root directory of your densepose: `ln -s $TORCH_PATH/lib /path/to/your/densepose/lib`~~

The issue deleted above has been solved by the official [Detectron](https://github.com/facebookresearch/Detectron/commit/df0e4972de432841fa0bf3a99ef86ac5b99471b7#diff-c9dcb56d00bdc377a5cde4915aba114f).

Please ensure that your Caffe2 installation and [protobuf installation](#google-protocol-buffers-protobuf) was successful before proceeding by running the following commands and checking their output as directed in the comments.

```bash
# To check if Caffe2 build was successful
$ python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
$ python -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
```

To avoid this [issue](https://github.com/facebookresearch/DensePose/issues/185) in case multiple CUDA libraries have been installed in your system and the symbol link `/usr/local/cuda` doesn't link to the specific version of CUDA that you're using, it is recommended to edit `$TORCH_PATH/share/cmake/Caffe2/Caffe2Target.cmake` in which replace `/usr/local/cuda/lib64/libculibos.a` with `/usr/local/cuda-x.x/lib64/libculibos.a`, otherwise it may trigger a linking error when [installing the custom DensePose operator](#build-the-custom-operators-library). **This issue only occurs in which the caffe2 was installed from binaries**.


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

Check that Detectron tests pass (e.g. for [`SpatialNarrowAsOp test`](tests/test_spatial_narrow_as_op.py)):

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

2. Download the source code of [PyTorch](https://github.com/pytorch/pytorch), then copy the folder `$PYTORCH/caffe2/utils/threadpool` into `$TORCH_PATH/lib/include/caffe2/utils/` (referred to this [issue](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-8-fatal-error-caffe2-utils-threadpool-ThreadPool-h-No-such-file-or-directory)).

3. Compile the custom operator:
    ```bash
    $ cd $DENSEPOSE/build
    $ cmake ..
    $ make
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
