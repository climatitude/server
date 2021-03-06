<!--
# Copyright (c) 2018-2020, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
-->

# Building Triton

Triton is built using the [build.py](../build.py) script. The build.py
script supports for the Docker build and a non-Docker build:

* [Build using Docker](#building-triton-with-docker) and the
  TensorFlow and PyTorch containers from [NVIDIA GPU Cloud
  (NGC)](https://ngc.nvidia.com>).

* [Build without Docker](#building-triton-without-docker).

## Building Triton with Docker

The easiest way to build Triton is to use Docker. The result of the
build will be a Docker image called *tritonserver* that will contain
the tritonserver executable in /opt/tritonserver/bin and the required
shared libraries in /opt/tritonserver/lib. The backend built for
Triton will be in /opt/tritonserver/backends (note that as of the
20.10 release the PyTorch and TensorRT backends are still included in
the core of Triton and so do not appear in
/opt/tritonserver/backends).

Building with Docker ensures that all the correct CUDA, cudnn,
TensorRT and other dependencies are handled for you. A Docker build is
enabled by using the --container-version flag with build.py. By
default no Triton features are enabled. The following build.py
invocation builds all features and backends.

```
$ ./build.py --version=<version> --container-version=<container version> --build-dir=/tmp/citritonbuild --enable-logging --enable-stats --enable-tracing --enable-metrics --enable-gpu-metrics --enable-gpu --filesystem=gcs --filesystem=s3 --endpoint=http --endpoint=grpc --repo-tag=common:<container tag> --repo-tag=core:<container tag> --repo-tag=backend:<container tag> --backend=custom --backend=ensemble --backend=tensorrt --backend=pytorch --backend=caffe2 --backend=identity:<container tag> --backend=repeat:<container tag> --backend=square:<container tag> --backend=onnxruntime:<container tag> --backend=tensorflow1:<container tag> --backend=tensorflow2:<container tag> --backend=python:<container tag> --backend=dali:<container tag>
```

Where <version> is the version to assign to Triton and <container
version> is the version to assign to the produced Docker
image. Typically you will set <version> to something meaningful for
your build and set <container version> to the value associated with
the Triton version found in the VERSION file. You can find these
associated values in CONTAINER_VERSION_MAP in build.py. For example,
if the VERSION file contents are "2.4.0dev" then the build invocation
should be:

```
$ ./build.py --version=0.0.0 --container-version=20.10dev ...
```

If you are building on master/main branch then <container tag> should
be set to "main". If you are building on a release branch you should
set the <container tag> to match. For example, if you are building on
the r20.09 branch you should set <container tag> to be "r20.09". If
can use a different <container tag> for a component to instead use the
corresponding branch/tag in the build. For example, if you have a
branch called "mybranch" in the identity_backend repo that you want to
use in the build, you would specify --backend=identity:mybranch.

By default build.py clones Triton repos from
https://github.com/triton-inference-server. Use the
--github-organization options to select a different URL.

The backends can also be built independently in each of the backend
repositories. See the [backend
repo](https://github.com/triton-inference-server/backend) for more
information.

## Building Triton without Docker

To build Triton without using Docker follow the [build.py steps
described above](#building-triton-with-docker) except do not specify
--container-version.

When building without Docker you must install the necessary CUDA
libraries and other dependencies needed for the build before invoking
build.py.

### CUDA, cuBLAS, cuDNN

For Triton to support NVIDIA GPUs you must install CUDA, cuBLAS and
cuDNN. These libraries must be installed on system include and library
paths so that they are available for the build. The version of the
libraries used in the Dockerfile build can be found in the [Framework
Containers Support
Matrix](https://docs.nvidia.com/deeplearning/frameworks/support-matrix/index.html).

For a given version of Triton you can attempt to build with
non-supported versions of the libraries but you may have build or
execution issues since non-supported versions are not tested.

### TensorRT

The TensorRT includes and libraries must be installed on system
include and library paths so that they are available for the
build. The version of TensorRT used in the Dockerfile build can be
found in the [Framework Containers Support
Matrix](https://docs.nvidia.com/deeplearning/frameworks/support-matrix/index.html).

For a given version of Triton you can attempt to build with
non-supported versions of TensorRT but you may have build or execution
issues since non-supported versions are not tested.

### TensorFlow

For instructions on how to build support for TensorFlow see the
[TensorFlow
backend](https://github.com/triton-inference-server/tensorflow_backend).

### ONNX Runtime

For instructions on how to build support for ONNX Runtime see the
[ONNX Runtime
backend](https://github.com/triton-inference-server/onnxruntime_backend)
and the CMakeLists.txt file contained in that repo. You must have a
version of the ONNX Runtime available on the build system and set the
TRITON_ONNXRUNTIME_INCLUDE_PATHS and TRITON_ONNXRUNTIME_LIB_PATHS
cmake variables appropriately.
