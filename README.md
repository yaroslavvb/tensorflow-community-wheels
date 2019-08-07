# tensorflow-community-wheels
TLDR; if you built a custom TensorFlow wheel, upload it somewhere, and post a link under Issues.
If you find a wheel useful, respond to the issue (ie, GitHub emoji), so that people know to keep maintaining that configuration.

Below is an example of building a wheel.

Before build you need to install [docker-ce](https://docs.docker.com/install/) and [nvidia-docker](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(Native-GPU-Support)).


# Docker powered build
```
# create directory for output wheel
mkdir -p ~/projects/tf

# pull docker image
docker pull tensorflow/tensorflow:devel-gpu-py3

# run docker
docker run --gpus all -it  -v ~/projects/tf:/my-devel tensorflow/tensorflow:devel-gpu-py3 bash

# fix bazel version
# (for TF v1.14 need bazel 0.25.2,
#  for TF v1.3 and lower - need bazel 0.21.0)
rm -rf /usr/local/bin/bazel
wget -O /bazel/installer.sh "https://github.com/bazelbuild/bazel/releases/download/0.25.2/bazel-0.25.2-installer-linux-x86_64.sh"
chmod +x /bazel/installer.sh
/bazel/installer.sh

# go to TF source folder
cd /tensorflow_src

# checkout to the required version
git checkout v1.14.0

# set some variables
# see full in Dockerfile: https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tools/dockerfiles/dockerfiles/devel-gpu.Dockerfile
export TF_ENABLE_XLA=1
export CC_OPT_FLAGS="-march=native -mno-avx"

# yes to all (enter, enter, enter, ...)
./configure

# run compilation (very long process)
bazel build --config=opt --config=cuda --noincompatible_strict_action_env //tensorflow/tools/pip_package:build_pip_package

# get wheel
bazel-bin/tensorflow/tools/pip_package/build_pip_package /my-devel/tensorflow_pkg1
```
