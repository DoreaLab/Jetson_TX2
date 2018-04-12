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

cmake -DCMAKE_BUILD_TYPE=Release -DPYTHON3_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.5m -DPYTHON_INCLUDE_DIR2=/usr/include/aarch64-linux-gnu/python3.5m -DPYTHON_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.5m.so -DPYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include -DCMAKE_INSTALL_PREFIX=/usr/local ..
