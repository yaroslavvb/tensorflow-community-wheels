# tensorflow-community-wheels
TLDR; if you built a custom TensorFlow wheel, upload is somewhere, and post a link under Issues.
If you find a wheel useful, respond to the issue (ie, GitHub emoji), so that people know to keep maintaining that configuration.

Below is an example of building/uploading a wheel

Configure is automated with https://github.com/yaroslavvb/stuff/blob/master/configure_tf.sh
Steps for configuring bazel env: https://medium.com/@yaroslavvb/setting-up-tensorflow-dev-environment-sep-19-fd27b321de14 (previous [version](https://github.com/tensorflow/tensorflow/issues/7443#issuecomment-279182613))

# Linux one-time build
```
tmux new-session -s bazel -n 0
cd ~/git2/tensorflow
source activate bazel

git fetch --all
git rebase tf/master

~/g/git/stuff/configure_tf.sh

export LD_LIBRARY_PATH="/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH"
export flags="--config=opt --config=cuda -k"
export tag=xlamem
export date=feb22

bazel build $flags -k //tensorflow/tools/pip_package:build_pip_package
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
mkdir -p ~/tfbins/$date.$tag
cp `find /tmp/tensorflow_pkg -type f ` ~/tfbins/$date.$tag
bazel test $flags //tensorflow/...
bazel test $flags -j 1 //tensorflow/...
bazel build $flags //tensorflow/...


export wheel=`find ~/tfbins/$date.$tag -type f`
export basename=`find ~/tfbins/$date.$tag -type f -printf "%f\n"`
cd ~/tfbins/$date.$tag
fullname=$date.$tag.$basename
ln -s $basename $fullname
export bucket=tensorflow-community-wheels
gsutil cp $fullname gs://$bucket
gsutil acl set public-read gs://$bucket/$fullname

echo https://storage.googleapis.com/tensorflow-community-wheels/$fullname
```
