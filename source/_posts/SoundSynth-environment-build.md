---
title: SoundSynth environment build
date: 2022-10-25 19:36:40
categories:
- SoundSynth
tags:
- environment
---

### SoundSynth

The project is from [https://github.com/ztzhang/SoundSynth](https://github.com/ztzhang/SoundSynth), this article is used to record the construction of the environment. 

Get "SoundSynth" by executing the following. 

```bash
git clone https://github.com/ztzhang/SoundSynth.git
```

### Environments

Ubuntu18.04 in VMware.

### Building Modal Sound

Most software can be installed by apt.

```bash
sudo apt install gcc
sudo apt install cmake
```

Except [GCC](https://gcc.gnu.org/) and [CMake](https://cmake.org/), I need to install [Eigen3](http://eigen.tuxfamily.org/index.php?title=Main_Page), [Qt5](https://doc.qt.io/qt-5/qt5-intro.html), [Protobuf](https://github.com/protocolbuffers/protobuf), [GSL](https://www.gnu.org/software/gsl/doc/html/index.html), [MKL](https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl.html) and Matlab.

#### Eigen3

Eigen3 is a header only library, so there is no building involved. 

```bash
wget https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.tar.gz
tar -xvf eigen-3.4.0.tar.gz
```

#### Auto install

Qt5, GSL and Boost will be installed after type the following command.

```bash
# Qt5
sudo apt install qt5-default
# GSL
sudo apt install libgsl0-dev
# Boost
sudo apt install libboost-all-dev
```

#### Protobuf

Install protobuf by source code.

```bash
wget https://github.com/protocolbuffers/protobuf/releases/download/v21.9/protobuf-cpp-3.21.9.tar.gz

tar -xzvf protobuf-cpp-3.21.9.tar.gz
cd protobuf
./configure --prefix=/usr/local/protobuf
sudo make && make install
sudo ldconfig
```

#### MKL

Set up the repository.

```bash
# download the key to system keyring
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

# add signed entry to apt sources and configure the APT client to use Intel repository:
echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
```

Update packages list and install MKL.

```bash
sudo apt update
sudo apt install intel-oneapi-mkl
```

#### Troubleshooting

After installation, specify theirs path ,run `./modal_sound/run_cmake.sh`. Three problems have arisen.

- Could not find MKL

  I have tried several ways to install MKL, but none of them worked. It seems cmake can not find the MKL. Finally, this problem was solved by add the following to **FindMKL.cmake**

  ```cmake
  find_path(MKL_INCLUDE_DIR mkl.h
      PATHS ENV INCLUDE
      PATHS ${SYSTEM_INC_PATH}
      PATHS $ENV{MKLROOT}/include  
      #new
      PATHS /opt/intel/oneapi/mkl/2022.2.0/include
  )
   
  find_library(MKL_LIBRARY mkl_core
      PATHS ENV LD_LIBRARY_PATH
      PATHS ${SYSTEM_LIB_PATH}
      PATHS ENV LIBRARY_PATH
      PATHS $ENV{MKLROOT}/lib
      # new
      PATHS /opt/intel/oneapi/mkl/2022.2.0/lib/intel64
  )
  ```

- Could NOT find Qt5Multimedia (missing: Qt5Multimedia_DIR)

  This problem is easy to solve, by contrast, just execute the following.

  ```bash
  sudo apt install qtmultimedia5-dev
  ```
  
- Unable to find the entry address of the thread function in the phread library.Add the following to the **CMakeLists.txt**

  ```makefile
  link_libraries(pthread)
  ```

### Building file generators

```bash
cd file_generators
chmod +x compile_cpps.sh
./compile_cpps.sh
```

### Installing BEM/FMM Solver

The project solve the Helmholtz equation related to sound propagation using a direct BEM method and an accelerated version using FMM.

#### BEM(not complete)

```bash
git clone -b release_1.1 git://last.hit.bme.hu/toolbox/nihu.git

cd nihu
mkdir build

MATLAB_PATH=/usr/local/MATLAB/R2022b
cmake ../src -DNIHU_INSTALL_DIR="../bin" -DNIHU_MATLAB_PATH=$MATLAB_PATH
make && make install
```

After installing, specify its path in **pre_compute/run_precalc_bem.sh**.

#### FMM

```bash
git clone https://github.com/zgimbutas/fmmlib3d.git
sudo apt-get install bison flex octave liboctave-dev mwrap

cd fmmlib3d
make mex
```

Samely, specify its path in **pre_compute/run_precalc.sh**.

### Building Bullet

```bash
cd bullet3/build3
./premake4_linux64 gmake

cd gamke
make App_RigidBodyFromObjExampleGui
```

