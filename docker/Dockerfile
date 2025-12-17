FROM ubuntu:19.10

MAINTAINER Bastian Bloessl mail@bastibl.net

RUN sed -i 's/archive.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update
RUN apt-get install -y git vim wget unzip sudo xserver-xorg openjdk-11-jre cmake \
    libtool build-essential pkg-config autogen ocaml ocamlbuild bison flex texinfo \
    python-dev python-mako python-six swig3.0 python3-mako python3-numpy python3-distutils \
    bzip2 tar

RUN groupadd -g 1000 -r android
RUN useradd -u 1000 -g 1000 --create-home -r android
RUN echo "android:android" | chpasswd
RUN echo "android ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-android
RUN usermod -aG sudo android
RUN usermod -aG plugdev android
USER android

WORKDIR /home/android/src

# clone repo WITHOUT recursive for now
RUN git clone https://github.com/bastibl/gnuradio-android.git

# Download a known good boost source & unpack
WORKDIR /home/android/src/gnuradio-android/Boost-for-Android
RUN wget https://boostorg.jfrog.io/artifactory/main/release/1.70.0/source/boost_1_70_0.tar.bz2
RUN tar xvjf boost_1_70_0.tar.bz2
RUN mv boost_1_70_0 boost

# Modify scripts to use local boost folder
RUN sed -i 's/1.69.0/boost/g' build-android.sh

# thrift build
WORKDIR /home/android/src/gnuradio-android/thrift
RUN git clean -xdf
RUN ./bootstrap.sh
RUN ./configure --disable-tests --disable-tutorial --with-cpp \
    --without-python --without-qt4 --without-qt5 --without-py3 \
    --without-go --without-nodejs --without-c_glib --without-php \
    --without-csharp --without-java --without-libevent --without-zlib
RUN make -j $(getconf _NPROCESSORS_ONLN)
RUN sudo make install

# Android toolchain
WORKDIR /home/android/src
RUN wget https://dl.google.com/android/repository/commandlinetools-linux-6200805_latest.zip
RUN unzip commandlinetools-linux-6200805_latest.zip

RUN mkdir -p /home/android/Android/Sdk
ENV ANDROID_HOME /home/android/Android/Sdk
WORKDIR /home/android/src/cmdline-tools/bin
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} --licenses
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "platforms;android-29"
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "ndk;21.3.6528147"
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "platform-tools"
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "build-tools;30.0.0"
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "cmake;3.10.2.4988404"
RUN yes | ./sdkmanager --sdk_root=${ANDROID_HOME} "cmdline-tools;latest"

# Now build gnuradio Android toolchain
WORKDIR /home/android/src/gnuradio-android
RUN ./build.sh --boost=boost
RUN ./build_aarch64.sh --boost=boost

ENTRYPOINT ["/bin/bash"]
