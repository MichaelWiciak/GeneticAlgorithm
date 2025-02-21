# BEAST

To install, compile and run BEAST on Linux (tested on Red Hat RHL9) follow the following steps:

git clone git@github.com:uol-school-of-computing/BEAST.git

cd BEAST

mkdir build

cd build

cmake ..

make -j 16

cd app

./beast ../projects/libdemos.so
