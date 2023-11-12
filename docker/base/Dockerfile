# Go cross compiler (xgo): Base cross-compilation layer
# Copyright (c) 2014 Péter Szilágyi. All rights reserved.
#
# Released under the MIT license.

FROM ubuntu:18.04

LABEL maintainer="techknowlogick <techknowlogick@gitea.io>"

# Mark the image as xgo enabled to support xgo-in-xgo
ENV XGO_IN_XGO 1

# Configure the Go environment, since it's not going to change
ENV PATH   /usr/local/go/bin:$PATH
ENV GOPATH /go

# Inject the remote file fetcher and checksum verifier
ADD fetch.sh /fetch.sh
ENV FETCH /fetch.sh
RUN chmod +x $FETCH

# address the cmake version issue
# RUN \
#   apt-get update && \
#   apt-get install -y wget make && \
#   apt install -y build-essential libssl-dev && \
#   wget https://github.com/Kitware/CMake/releases/download/v3.16.5/cmake-3.16.5.tar.gz && \
#   tar -zxvf cmake-3.16.5.tar.gz && \
#   cd cmake-3.16.5 && \
#   ./bootstrap && \
#   make -j6 && \
#   make install && \
#   cmake --version 

  # wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null && \
  # echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ bionic main' | tee /etc/apt/sources.list.d/kitware.list >/dev/null && \
  # rm /usr/share/keyrings/kitware-archive-keyring.gpg && \
  # apt-get install -y kitware-archive-keyring && \
  # apt-get update && \
  # apt-get install -y cmake

# Make sure apt-get is up to date and dependent packages are installed

RUN \
  apt-get update && \
  apt-get install -y wget make && \
  apt install -y build-essential libssl-dev && \
  wget https://github.com/Kitware/CMake/releases/download/v3.16.5/cmake-3.16.5.tar.gz && \
  tar -zxvf cmake-3.16.5.tar.gz && \
  cd cmake-3.16.5 && \
  ./bootstrap && \
  make -j6 && \
  make install && \
  cd .. && \
  apt-get install -y automake autogen build-essential ca-certificates                    \
    gcc-5-arm-linux-gnueabi g++-5-arm-linux-gnueabi libc6-dev-armel-cross                \
    gcc-5-arm-linux-gnueabihf g++-5-arm-linux-gnueabihf libc6-dev-armhf-cross            \
    gcc-5-aarch64-linux-gnu g++-5-aarch64-linux-gnu libc6-dev-arm64-cross                \
    gcc-5-mips-linux-gnu g++-5-mips-linux-gnu libc6-dev-mips-cross                       \
    gcc-5-mipsel-linux-gnu g++-5-mipsel-linux-gnu libc6-dev-mipsel-cross                 \
    gcc-5-mips64-linux-gnuabi64 g++-5-mips64-linux-gnuabi64 libc6-dev-mips64-cross       \
    gcc-5-mips64el-linux-gnuabi64 g++-5-mips64el-linux-gnuabi64 libc6-dev-mips64el-cross \
    gcc-5-multilib g++-5-multilib gcc-mingw-w64 g++-mingw-w64                            \
    gcc-6-arm-linux-gnueabi g++-6-arm-linux-gnueabi libc6-dev-armel-cross                \
    gcc-6-arm-linux-gnueabihf g++-6-arm-linux-gnueabihf libc6-dev-armhf-cross            \
    gcc-6-aarch64-linux-gnu g++-6-aarch64-linux-gnu libc6-dev-arm64-cross                \
    gcc-6-mips-linux-gnu g++-6-mips-linux-gnu libc6-dev-mips-cross                       \
    gcc-6-mipsel-linux-gnu g++-6-mipsel-linux-gnu libc6-dev-mipsel-cross                 \
    gcc-6-mips64-linux-gnuabi64 g++-6-mips64-linux-gnuabi64 libc6-dev-mips64-cross       \
    gcc-6-mips64el-linux-gnuabi64 g++-6-mips64el-linux-gnuabi64 libc6-dev-mips64el-cross \
    gcc-6-s390x-linux-gnu g++-6-s390x-linux-gnu libc6-dev-s390x-cross \
    gcc-6-powerpc64le-linux-gnu g++-6-powerpc64le-linux-gnu libc6-dev-powerpc-ppc64-cross \
    gcc-6-multilib g++-6-multilib gcc-7-multilib g++-7-multilib gcc-mingw-w64 g++-mingw-w64 \
    clang llvm-dev libtool libxml2-dev uuid-dev libssl-dev swig openjdk-8-jdk pkg-config patch \
    xz-utils cpio zip unzip p7zip git mercurial bzr texinfo help2man curl mercurial && \
    apt-get install -y libssl1.1 openssl ca-certificates && \
    update-ca-certificates && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  find /var/log -type f | while read f; do echo -ne '' > $f; done;

# Fix any stock package issues
RUN ln -s /usr/include/asm-generic /usr/include/asm

# Add patches directory for patching later
ADD patches /patches

##########################
# Darwin Toolchain build #
##########################

# # Configure the container for OSX cross compilation
# ENV OSX_SDK MacOSX11.1.sdk
# ENV OSX_SDK_PATH https://github.com/phracker/MacOSX-SDKs/releases/download/11.0-11.1/${OSX_SDK}.tar.xz

# # Download the osx sdk and build the osx toolchain
# # We download the osx sdk, patch it and pack it again to be able to throw the patched version at osxcross
# RUN \
#   $FETCH $OSX_SDK_PATH 9b86eab03176c56bb526de30daa50fa819937c54b280364784ce431885341bf6 && \
#   tar -xf `basename $OSX_SDK_PATH` && rm -f `basename $OSX_SDK_PATH`
# ADD patch.tar.xz $OSX_SDK/usr/include/c++
# RUN tar -cf - $OSX_SDK/ | xz -c - > $OSX_SDK.tar.xz && rm -rf $OSX_SDK

# ENV OSX_NDK_CROSS /usr/local/osx-ndk-cross

# # Actually build the toolchain
# RUN  \
#   git clone https://github.com/tpoechtrager/osxcross.git                                          && \
#   cd osxcross && git checkout 0f87f567dfaf98460244471ad6c0f4311d62079c && cd ../                  && \
#   mv $OSX_SDK.tar.xz /osxcross/tarballs/                                                          && \
#   OSX_VERSION_MIN=10.13 UNATTENDED=1 LD_LIBRARY_PATH=/osxcross/target/lib /osxcross/build.sh      && \
#   mv /osxcross/target $OSX_NDK_CROSS                                                              && \
#   rm -rf /osxcross

# # Make libxar known to the ld64 and cctools build
# ENV LD_LIBRARY_PATH=$OSX_NDK_CROSS/lib

# ENV PATH $OSX_NDK_CROSS/bin:$PATH

# # Configure the container for iOS cross compilation
# ENV IOS_NDK_ARM_7     /usr/local/ios-ndk-arm-7
# ENV IOS_NDK_ARM64     /usr/local/ios-ndk-arm64
# ENV IOS_SIM_NDK_AMD64 /usr/local/ios-sim-ndk-amd64

# ADD update_ios.sh /update_ios.sh
# ENV UPDATE_IOS /update_ios.sh
# RUN chmod +x $UPDATE_IOS
# ENV IOS_SDK_VERSION iPhoneOS12.4
# RUN \
#   IOS_SDK_FILE=$IOS_SDK_VERSION.sdk.zip && \
#   IOS_SDK_PATH=http://resources.airnativeextensions.com/ios/$IOS_SDK_FILE && \
#   $FETCH $IOS_SDK_PATH 365818df5aea857e19914f577694f80ac864a68c && \
#   mv `basename $IOS_SDK_PATH` $IOS_SDK_VERSION.sdk.tmp.zip && \
#   $UPDATE_IOS /$IOS_SDK_VERSION.sdk.tmp.zip

# # 2 separate steps to make it easier to debug
# RUN \
#   rm -rf /$IOS_SDK_VERSION.sdk.tmp.zip  && \
#   # TODO the new iPhoneOS version does not have the /include/c++ directory. Hope it's not an issue...
#   # if it becomes an issue, download from here https://github.com/phracker/MacOSX-SDKs/tree/master/MacOSX10.13.sdk/usr/include
#   # the previous version. Maybe it's not going to be an issue.
#   mkdir -p $IOS_NDK_ARM64/SDK/$IOS_SDK_VERSION.sdk/usr/include/c++ && \
#   cp -r $OSX_NDK_CROSS/SDK/$OSX_SDK/usr/include/c++/v1 $IOS_NDK_ARM64/SDK/$IOS_SDK_VERSION.sdk/usr/include/c++

# ENV PATH $IOS_NDK_ARM64/bin:$PATH

# Configure the container for Android cross compilation
# android-ndk-r21e c3ebc83c96a4d7f539bd72c241b2be9dcd29bda9
# android-ndk-r22b 9ece64c7f19763dd67320d512794969930fce9dc
ENV ANDROID_NDK         android-ndk-r22b
ENV ANDROID_NDK_PATH    http://dl.google.com/android/repository/$ANDROID_NDK-linux-x86_64.zip
ENV ANDROID_NDK_ROOT    /usr/local/$ANDROID_NDK
ENV ANDROID_NDK_LIBC    $ANDROID_NDK_ROOT/sources/cxx-stl/llvm-libc++
ENV ANDROID_PLATFORM    22
ENV ANDROID_CHAIN_ARM   arm-linux-androideabi-4.9
ENV ANDROID_CHAIN_ARM64 aarch64-linux-android-4.9
ENV ANDROID_CHAIN_AMD64 x86_64-4.9
ENV ANDROID_CHAIN_386   x86-4.9

RUN \
  $FETCH $ANDROID_NDK_PATH 9ece64c7f19763dd67320d512794969930fce9dc

RUN \
  unzip `basename $ANDROID_NDK_PATH` \
    "$ANDROID_NDK/build/*"                                           \
    "$ANDROID_NDK/sources/cxx-stl/llvm-libc++/include/*"       \
    "$ANDROID_NDK/sources/cxx-stl/llvm-libc++/libs/armeabi*/*" \
    "$ANDROID_NDK/sources/cxx-stl/llvm-libc++/libs/arm64*/*"   \
    "$ANDROID_NDK/sources/cxx-stl/llvm-libc++/libs/x86_64/*"   \
    "$ANDROID_NDK/sources/cxx-stl/llvm-libc++/libs/x86/*"      \
    "$ANDROID_NDK/prebuilt/*"                           \
#    "$ANDROID_NDK/platforms/*/arch-arm/*"                            \
#    "$ANDROID_NDK/platforms/*/arch-arm64/*"                          \
#    "$ANDROID_NDK/platforms/*/arch-x86_64/*"                         \
#    "$ANDROID_NDK/platforms/*/arch-x86/*"                            \
    "$ANDROID_NDK/toolchains/llvm/*"                                 \
    "$ANDROID_NDK/toolchains/$ANDROID_CHAIN_ARM/*"                   \
    "$ANDROID_NDK/toolchains/$ANDROID_CHAIN_ARM64/*"                 \
    "$ANDROID_NDK/toolchains/$ANDROID_CHAIN_386/*"                   \
    "$ANDROID_NDK/toolchains/$ANDROID_CHAIN_AMD64/*" -d /usr/local > /dev/null && \
  rm -f `basename $ANDROID_NDK_PATH`

ENV PATH /usr/$ANDROID_CHAIN_ARM/bin:$PATH
ENV PATH /usr/$ANDROID_CHAIN_ARM64/bin:$PATH
ENV PATH /usr/$ANDROID_CHAIN_386/bin:$PATH
ENV PATH /usr/$ANDROID_CHAIN_AMD64/bin:$PATH

# setup Android SDK tooling too
ENV ANDROID_SDK_PATH https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip
ENV ANDROID_SDK_ROOT /usr/local/android-sdk

RUN \
    $FETCH $ANDROID_SDK_PATH 87f6dcf41d4e642e37ba03cb2e387a542aa0bd73cb689a9e7152aad40a6e7a08 && \
    unzip `basename $ANDROID_SDK_PATH` \
        -d $ANDROID_SDK_ROOT > /dev/null && \
    rm -f `basename $ANDROID_SDK_PATH`

RUN \
    mv $ANDROID_SDK_ROOT/cmdline-tools $ANDROID_SDK_ROOT/tools && \
    mkdir $ANDROID_SDK_ROOT/cmdline-tools && \
    mv $ANDROID_SDK_ROOT/tools $ANDROID_SDK_ROOT/cmdline-tools/

RUN \
    echo "Y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "platforms;android-22" "platforms;android-21" "platforms;android-16"

# Inject the new Go root distribution downloader and bootstrapper
ADD bootstrap_pure.sh /bootstrap_pure.sh
ENV BOOTSTRAP_PURE /bootstrap_pure.sh
RUN chmod +x $BOOTSTRAP_PURE

# Inject the C dependency cross compiler
ADD build_deps.sh /build_deps.sh
ENV BUILD_DEPS /build_deps.sh
RUN chmod +x $BUILD_DEPS

# Inject the container entry point, the build script
ADD build.sh /build.sh
ENV BUILD /build.sh
RUN chmod +x $BUILD

ENTRYPOINT ["/build.sh"]
