# Installing DensePose

Installing DensePose is not an easy thing except building it from the provided [`Dockerfile`](https://github.com/facebookresearch/DensePose/blob/master/docker/Dockerfile). Here, I provided an installation guide based on the official [DensePose Installation](https://github.com/facebookresearch/DensePose/blob/master/INSTALL.md) and an [installation guide](http://linkinpark213.com/2018/11/18/densepose-minesweeping/) provided by [@linkinpark213](https://github.com/linkinpark213).


**Requirements:**

- NVIDIA GPU, Linux, **Python2** (It is not recommended to use Python3)
- Caffe2, various standard Python packages, and the COCO API; Instructions for installing these dependencies are found below
- [cmake 3.8.2](https://cmake.org/files/). (**Do not use the latest version**.)
- [GCC-4.9.2](https://gcc.gnu.org/mirrors.html) (Because `libcaffe2.so` installed from `conda` was pre-compiled using exactly this version of gcc)
- [protobuf-3.5.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.5.0) (this version is exactly specified in the header of `caffe2.pb.h` of caffe2 installed from `conda`)

**Notes:**

- Detectron operators currently do not have CPU implementation; a GPU system is required.
- My system configurations:
  * System: CentOS 7.0
  * GPU: NVIDIA Tesla P100
  * CUDA 9.0 with cuDNN 7.4.2
  * Python 2.7.15 environment created by `conda create -n py2 python=2.7`, set `CONDA_ENV_PATH=/path/to/my/conda/envs/py2`

## Installing required python packages (dependencies)
- PyTorch dependencies: `conda install numpy pyyaml mkl mkl-include setuptools cffi typing`
- COCOAPI dependencies: `conda install cython matplotlib`
- Caffe2 dependencies: `conda install pydot future networkx`
- DensePose dependencies: `conda install opencv mock scipy h5py memory_profiler`
- [SMPL](http://smpl.is.tue.mpg.de/) model dependencies: `pip install chumpy`

## Installing Caffe2 (from PyTorch)
[Caffe2](https://caffe2.ai/) has been merged and integrated into [PyTorch](https://pytorch.org/), which makes its easier to be installed by the following command:

`conda install pytorch torchvision cudatoolkit=9.0 -c pytorch`

By this command, the latest stable version of [PyTorch](https://pytorch.org/) (currently, PyTorch-1.0.1) and [Caffe2](https://caffe2.ai/) will be installed under the conda site-packages directory, set `TORCH_PATH=$CONDA_ENV_PATH/lib/python2.7/site-packages/torch`.

Please note that the command above is for `CUDA 9.0` on Linux system, you can select your preferences and run the install command from [PyTorch - Get Started](https://pytorch.org/get-started/locally/).

Adjust your `PYTHONPATH` environment variable to include this location: `$TORCH_PATH/lib` (referred to the [issue: Detectron ops lib not found](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-2-Detectron-ops-lib-not-found)). If this doesn't work when [setting up python modules of densepose](#densepose), try creating a symbol link to this path in the root directory of your densepose: `ln -s $TORCH_PATH/lib /path/to/your/densepose/lib`

Please ensure that your Caffe2 installation was successful before proceeding by running the following commands and checking their output as directed in the comments.

```bash
# To check if Caffe2 build was successful
$ python2 -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
$ python2 -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
```

To avoid this [issue](https://github.com/facebookresearch/DensePose/issues/185) in case multiple CUDA libraries have been installed in your system and the symbol link `/usr/local/cuda` doesn't link to the specific version of CUDA that you're using, it is recommended to edit `$TORCH_PATH/share/cmake/Caffe2/Caffe2Target.cmake` in which replace `/usr/local/cuda/lib64/libculibos.a` with `/usr/local/cuda-x.x/lib64/libculibos.a`, otherwise it may trigger a linking error when [installing the custom DensePose operator]().

## Other Dependencies

- Install the [COCO API](https://github.com/cocodataset/cocoapi):

    ```bash
    # COCOAPI=/path/to/clone/cocoapi
    $ git clone https://github.com/cocodataset/cocoapi.git $COCOAPI
    $ cd $COCOAPI/PythonAPI
    # Install into global site-packages
    $ make install
    ```

    Note that instructions like `# COCOAPI=/path/to/install/cocoapi` indicate that you should pick a path where you'd like to have the software cloned and then set an environment variable (`COCOAPI` in this case) accordingly.

- Install [GCC-4.9.2](https://ftp.gnu.org/gnu/gcc/gcc-4.9.2/)
    
    Before installing `GCC`, you need to make sure which exactly version of GCC was used to build caffe2 in the precompiled conda package. 
    
    So far, running command `strings -a $TORCH_PATH/lib/libcaffe2.so | grep "GCC: ("` will give the output:
    `GCC: (GNU) 4.9.2 20150212 (Red Hat 4.9.2-6)`

    ```bash
    $ mkdir /path/to/gcc-4.9.2/build && cd /path/to/gcc-4.9.2/build
    $ ../configure --prefix=/path/to/build --enable-checking=release --enable-languages=c,c++ --disable-multilib
    $ make
    $ make install
    $ cd bin/
    $ ln -s gcc cc  # create a symbol link 'cc' for 'gcc'
    $ cp -r include/c++ $CONDA_ENV_PATH/include
    ```

    After installing gcc-4.9.2 successfully, you need to adjust your `$PATH` and `$LD_LIBRARY_PATH` environment variables for `build/bin` and `build/lib64`.

- Install [protobuf](https://github.com/protocolbuffers/protobuf/tags)

    Before installing `protobuf`, you need to make sure which exactly version of `protobuf` specified by your installed `caffe2`. You can find this information in the header of `$TORCH_PATH/lib/include/caffe2/proto/caffe2.pb.h`:

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

    Now, let's first install [protobuf-3.5.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.5.0) C/C++ API:
    ```bash
    $ mkdir /path/to/protobuf-3.5.0/build && cd /path/to/protobuf-3.5.0/build
    $ ../configure --prefix=/path/to/build CXXFLAGS="-fPIC"
    $ make
    $ make install
    ```
    
    After installing protobuf C/C++ API, you need to adjust the `$PATH` and `$LD_LIBRARY_PATH` for `build/bin` and `build/lib`, as aforementioned in the installation of GCC. Or you can put these built files into your conda environment which will only take effect within the environment (which I did in this way):

    ```bash
    $ cd /path/to/protobuf-3.5.0/build
    $ cp bin/protoc $CONDA_ENV_PATH/bin
    $ cp -r include/google $CONDA_ENV_PATH/include
    $ cp lib/libproto* $CONDA_ENV_PATH/lib
    ```
    
    Install Python API of protobuf (before that, ensure that you're in the correct python environment)
    
    ```bash
    $ cd /path/to/protobuf-3.5.0/python
    $ python2 setup.py build
    $ python2 setup.py install
    ```


## Densepose

Clone the Densepose repository:

```bash
# DENSEPOSE=/path/to/clone/densepose
$ git clone https://github.com/facebookresearch/densepose $DENSEPOSE
```

Set up Python modules:

```bash
$ cd $DENSEPOSE
$ python2 setup.py develop
```

Check that Detectron tests pass (e.g. for [`SpatialNarrowAsOp test`](tests/test_spatial_narrow_as_op.py)):

```bash
$ python2 $DENSEPOSE/detectron/tests/test_spatial_narrow_as_op.py
```

**Build the custom operators library** (this part took me too too much time):

1. Edit `$DENSEPOSE/CMakeLists.txt` (Thanks to [@hyousamk's solution](https://github.com/facebookresearch/DensePose/issues/119)), you download [`CMakeLists.txt`](https://github.com/Johnqczhang/densepose_installation/blob/master/CMakeLists.txt) from this repository into your `$DENSEPOSE`, then replace corresponding paths specified in this file with yours:
    
    ```diff
    diff --git a/CMakeLists.txt b/CMakeLists.txt
    index 488ea86..b59d9bb 100644
    --- a/CMakeLists.txt
    +++ b/CMakeLists.txt
    @@ -1,11 +1,24 @@
    cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

    +# set caffe2 cmake path manually
    +set(Caffe2_DIR "/path/to/conda/envs/py2/lib/python2.7/site-packages/torch/share/cmake/Caffe2")
    +# set cuDNN path
    +set(CUDNN_INCLUDE_DIR "/path/to/your/cudnn/include")
    +set(CUDNN_LIBRARY "/path/to/your/libcudnn/libcudnn.so")
    +include_directories("/path/to/conda/envs/py2/include")
    +# add static protobuf library
    +add_library(libprotobuf STATIC IMPORTED)
    +set(PROTOBUF_LIB "/path/to/conda/envs/py2/lib/libprotobuf.a")
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

2. Download [pytorch](https://github.com/pytorch/pytorch) source code, then copy the folder `$PYTORCH/caffe2/utils/threadpool` into `$TORCH_PATH/lib/include/caffe2/utils/` (referred to this [issue](http://linkinpark213.com/2018/11/18/densepose-minesweeping/#2-8-fatal-error-caffe2-utils-threadpool-ThreadPool-h-No-such-file-or-directory)).

3. Compile the custom operator:
    
    ```bash
    $ cd $DENSEPOSE/build
    $ cmake ..
    $ make
    ```

4. Check that the custom operator tests pass:

    ```bash
    $ python2 $DENSEPOSE/detectron/tests/test_zero_even_op.py
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

Congratulations! Now you've installed DensePose successfully! :tada:

Happy Hunting DensePose!

## Acknowledgements
- [DensePose Github](https://github.com/facebookresearch/DensePose)
- [[MineSweeping] The Long Struggle of DensePose Installation](http://linkinpark213.com/2018/11/18/densepose-minesweeping/), a very helpful post about the DensePose installation provided by [@linkinpark213](https://github.com/linkinpark213).
- Special thanks to [@tete1030](https://github.com/tete1030) for his great help in debugging the installation of custom operator with gcc compiler.
