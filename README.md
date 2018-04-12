# Jetson_TX2
sudo add-apt-repository universe

sudo apt-get update -y

sudo apt-get install python3-pip

sudo apt-get install python3-numpy

sudo apt-get install libfreetype6-dev

sudo apt-get install pkg-config

sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-devlibhdf5-serial-dev protobuf-compiler -y

sudo apt-get install libatlas-base-dev -y


## Check OpenCV
pkg-config --libs --cflags opencv

https://www.learnopencv.com/install-opencv3-on-ubuntu/

https://docs.opencv.org/3.3.0/d7/d9f/tutorial_linux_install.html

https://github.com/jetsonhacks/buildOpenCVTX2/blob/master/buildOpenCV.sh


cmake -DCMAKE_BUILD_TYPE=Release -DPYTHON3_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.5m -DPYTHON_INCLUDE_DIR2=/usr/include/aarch64-linux-gnu/python3.5m -DPYTHON_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.5m.so -DPYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include -DCMAKE_INSTALL_PREFIX=/usr/local ..

## Tensorflow
The official instructions on building TensorFlow are here: https://www.tensorflow.org/install/install_sources

## Prerequisites

We are assuming a build with CUDA support, as well as including SIMD optimizations (SSE3, SSE4, AVX, AVX2, FMA), on a
Debian-like system (e.g. Ubuntu Linux).

On new systems, one will have to install CUDA, CuDNN, plus the following dependencies:

    $ sudo apt-get install python3-numpy python3-dev python3-pip python3-wheel libcupti-dev

(Leave out `libcupti-dev` when not building with GPU support.)

Good to know: The compute capabilities for 
* Maxwell TITAN X: `5.2`
* Pascal TITAN X (2016): `6.1`
* GeForce GTX 1080 Ti: `6.1`

(See [here](https://developer.nvidia.com/cuda-gpus) for the full list.)

## Installing Bazel

Bazel is Google's own build system, required to build TensorFlow. Building TensorFlow usually requires an up-to-date
version of Bazel; there is a good chance that whatever your package manager provides will be outdated.

There are various ways to obtain a build of Bazel (see https://bazel.build/versions/master/docs/install-ubuntu.html).

### Option 1: Using the Bazel APT repository

Recommended by Google, but you need to be comfortable adding another APT package souce.

      $ echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
      $ curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
      $ sudo apt-get update && sudo apt-get install bazel

### Option 2: Building from source

In case you prefer building from source, it's unfortunately not as easy as cloning the Git repository and typing `make`.
Recent versions of Bazel can only be built with Bazel, unless one downloads a distribution source build, which contains
some already pre-generated files. With one such installation in place, one could build Bazel straight from the
repository source, but that's probably not necessary.

So we will go with building a distribution build, which is reasonably straightforward:

* Download a distribution package from the releases page. The current version at the time of writing was 0.11.1.

      $ mkdir bazel && cd bazel
      $ wget https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel-0.11.1-dist.zip

* Unzip the sources. This being a zip file, the files are stored without containing folder. Glad we already put it in
its own directory...

      $ unzip bazel-0.11.1-dist.zip

* Compile Bazel

      $ bash ./compile.sh

* The output executable is now located in `output/bazel`. Add a `PATH` entry to your `.bashrc`, or just export it in
your current shell:

      $ export PATH=`pwd`/output:$PATH

You should now be able to call the `bazel` executable from anywhere on your filesystem.

## Installing TensorFlow

### Building TensorFlow

* Create a Python 3 virtualenv, if you have not done this yet. For example:

      $ virtualenv -p python3 --system-site-packages ~/.virtualenvs/tf_dev

* Activate your respective Python 3 based virtual environment.

      $ source ~/.virtualenvs/tf_dev/bin/activate
      
  * This can later be deactivated with

        $ deactivate

* Clone the sources, and check out the desired branch. At the time of writing, 1.7.0 was the latest version; adjust if
necessary.

      $ git clone https://github.com/tensorflow/tensorflow
      $ cd tensorflow
      $ git checkout v1.7.0

* Run the configuration script

      $ ./configure

    You can leave most defaults, but do specify the following (or similar):

      CUDA support -> Y
      CUDA compute capability -> 5.2,6.1

* Compile TensorFlow using Bazel.

  This command will build Tensorflow using optimized settings for the current machine architecture.

      $ bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

    * Leave out `--config=cuda` when not building with GPU support.
    
    * TensorFlow 1.5+: As [mentioned here](https://github.com/tensorflow/tensorflow/issues/16694), you might have to add
    `--action_env=LD_LIBRARY_PATH=/path/to/cuda/lib64/stubs:${LD_LIBRARY_PATH}` to the call to Bazel, in case you encounter
    linking errors related to CUDA/cuDNN.
    
    * Due to [this issue](https://github.com/tensorflow/tensorflow/issues/15492), you might need to add the argument
    `--incompatible_load_argument_is_label=false` if you see an error similar to
    `name 'sycl_library_path' is not defined`.

    * Add `-c dbg --strip=never` in case you do not want debug symbols to be stripped (e.g. for debugging purposes).
    Usually, you won't need to add this option.
    
    * Add `--compilation_mode=dbg` to build in debug instead of release mode, i.e. without optimizations.
    You shouldn't do this unless you really want to.
    
* We still need to build a Python package using the now generated build_pip_package script.

      $ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

* Now there will be a package inside /tmp/tensorflow_pkg, which can be installed with pip.

      $ pip install /tmp/tensorflow_pkg/tensorflow-1.7.0-cp35-cp35m-linux_x86_64.whl

    (Adjust the file name, if necessary.)

That should be it!

### Notes on building with unsupported system

* Additional steps may be necessary when building TensorFlow on an officially unsupported system (e.g. on a non-LTS
Ubuntu version). For example, Ubuntu versions >16.04 ship with GCC 6 as default compiler, but CUDA 8 (still) requires
a GCC compiler from the GCC 5.x series. In this case:

    * Build GCC 5.x from source and install, preferably user-local. (https://github.com/kmhofmann/build_stuff provides
    a script to build various versions of GCC from source.)

      Add the respective paths to the `PATH` and `LD_LIBRARY_PATH` variables, e.g.:
    
          $ export PATH=$HOME/local/bin:$PATH
          $ export LD_LIBRARY_PATH=$HOME/local/lib64:$LD_LIBRARY_PATH
        
      Ensure that `gcc --version` is the desired version.
      
    * Proceed with building, as described below.
