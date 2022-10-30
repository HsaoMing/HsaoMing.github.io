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

### Environments

Ubuntu18.04 in VMware.

### Build

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

#### Qt5

[Qt5](https://wiki.qt.io/Install_Qt_5_on_Ubuntu) will be installed after type the following command.

```bash
sudo apt install qt5-default
```

#### Protobuf

To build [protobuf](https://developers.google.cn/protocol-buffers/) from source, [bazel](https://bazel.build/), git and g++ are needed.

Commonly, git and g++ are already installed. 

Download file from [https://github.com/bazelbuild/bazel/releases](https://github.com/bazelbuild/bazel/releases). 

```bash
wget https://github.com/bazelbuild/bazel/releases/download/4.2.3/bazel-4.2.3-installer-linux-x86_64.sh

./bazel-version-installer-linux-x86_64.sh --user
```



https://github.com/bazelbuild/bazel/releases/download/4.2.3/bazel-4.2.3-installer-linux-x86_64.sh

https://github.com/protocolbuffers/protobuf/releases/download/v21.8/protobuf-cpp-3.21.8.tar.gz

