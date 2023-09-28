# https://www.learnopencv.com/install-opencv3-on-ubuntu/
# https://www.osradar.com/how-to-install-opencv-on-ubuntu-20-04/

# ============== BASE IMAGE =====================
# ===============================================

ARG UBUNTU_VERSION=22.04

FROM ubuntu:${UBUNTU_VERSION} AS base

LABEL maintainer="https://github.com/Borda"

ARG PYTHON_VERSION=3.9
ARG OPENCV_VERSION=4.8.0

# Needed for string substitution
SHELL ["/bin/bash", "-c"]
# https://techoverflow.net/2019/05/18/how-to-fix-configuring-tzdata-interactive-input-when-building-docker-images/
ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Europe/Prague

RUN \
    # add sources for older pythons
    apt-get update -q --fix-missing && \
    apt-get install -y --no-install-recommends software-properties-common && \
    if [[ "$PYTHON_VERSION" =~ ^(2.7|3.6|3.7)$ ]]; then \
        add-apt-repository ppa:deadsnakes/ppa ; \
    fi && \
    apt-add-repository universe && \
    apt-get -y update -qq --fix-missing && \
    apt-get -y install --no-install-recommends \
        build-essential \
        cmake \
        ffmpeg \
        $( [ "$UBUNTU_VERSION" = "20.04" ] && libtbb2 || libtbb12 ) \
        gfortran \
        apt-utils \
        pkg-config \
        qtbase5-dev qt5-qmake \
        libopenblas-base \
        libgstreamer1.0 \
        libavcodec58 \
        libavformat58 \
        libswscale5 \
        libdc1394-22 \
        libjpeg8 \
        libtiff5 \
        liborc-0.4 \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-dev \
        $( [ ${PYTHON_VERSION%%.*} -ge 3 ] && echo "python${PYTHON_VERSION}-distutils" ) \
        curl \
        wget \
        unzip \
    && \
    rm -rf /var/lib/apt/lists/* && \

    # Set the default python and install PIP packages
    update-alternatives --install /usr/bin/python${PYTHON_VERSION%%.*} python${PYTHON_VERSION%%.*} /usr/bin/python${PYTHON_VERSION} 1 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 1 && \

    # install python dependencies
    # sysctl -w net.ipv4.ip_forward=1 ; \
    if [[ "$PYTHON_VERSION" =~ ^(2.7|3.6)$ ]]; then \
        PIP_URL="https://bootstrap.pypa.io/pip/$PYTHON_VERSION/get-pip.py" ; \
    else \
        apt-get -y install --no-install-recommends python${PYTHON_VERSION}-distutils ; \
        PIP_URL="https://bootstrap.pypa.io/get-pip.py" ; \
    fi && \
    curl $PIP_URL | python && \
    pip install numpy


# ============ COMPILE OPENCV ===================
# ===============================================

FROM base AS builder

RUN \
    # Install all dependencies for OpenCV
    apt-get -y update -qq --fix-missing && \
    apt-get -y install --no-install-recommends \
        checkinstall \
        libopenblas-dev \
        liblapack-dev \
        libatlas-base-dev \
        libavcodec-dev \
        libavformat-dev \
        libavutil-dev \
        libswscale-dev \
        libjpeg8-dev \
        libpng-dev \
        libtiff5-dev \
        libdc1394-22-dev \
        libxine2-dev \
        libv4l-dev \
        liborc-0.4-dev \
        libgstreamer1.0-dev \
        libgstreamer-plugins-base1.0-0 \
        libgstreamer-plugins-base1.0-dev \
        libglew-dev \
        libpostproc-dev \
        libeigen3-dev \
        libtbb-dev \
        zlib1g-dev \
        libsm6 \
        libxext6 \
        libxrender1

RUN \
    # Download OpenCV
    wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip \
      -O opencv.zip --progress=bar:force:noscroll --no-check-certificate && \
    unzip -q opencv.zip && \
    mv /opencv-${OPENCV_VERSION} /opencv && \
    rm opencv.zip && \
    wget https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip \
      -O opencv_contrib.zip --progress=bar:force:noscroll --no-check-certificate && \
    unzip -q opencv_contrib.zip && \
    mv /opencv_contrib-${OPENCV_VERSION} /opencv_contrib && \
    rm opencv_contrib.zip

RUN \
    # Prepare build
    mkdir /opencv/build && \
    cd /opencv/build && \
    cmake \
      -D CMAKE_BUILD_TYPE=RELEASE \
      -D BUILD_PYTHON_SUPPORT=ON \
      -D BUILD_DOCS=ON \
      -D BUILD_PERF_TESTS=OFF \
      -D BUILD_TESTS=OFF \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
      -D BUILD_opencv_python3=$( [ ${PYTHON_VERSION%%.*} -ge 3 ] && echo "ON" || echo "OFF" ) \
      -D BUILD_opencv_python2=$( [ ${PYTHON_VERSION%%.*} -lt 3 ] && echo "ON" || echo "OFF" ) \
      -D PYTHON${PYTHON_VERSION%%.*}_EXECUTABLE=$(which python${PYTHON_VERSION}) \
      -D PYTHON_DEFAULT_EXECUTABLE=$(which python${PYTHON_VERSION}) \
      -D BUILD_EXAMPLES=OFF \
      -D WITH_IPP=OFF \
      -D WITH_FFMPEG=ON \
      -D WITH_GSTREAMER=ON \
      -D WITH_V4L=ON \
      -D WITH_LIBV4L=ON \
      -D WITH_TBB=ON \
      -D WITH_QT=ON \
      -D WITH_OPENGL=ON \
      -D WITH_LAPACK=ON \
      -D ENABLE_PRECOMPILED_HEADERS=ON \
      -d BUILD_SHARED_LIBS=OFF \
      #-D WITH_HPX=ON \
      ..

RUN \
    # Build the package
    cd /opencv/build && \
    make -j$(nproc)

# ============= FINAL IMAGE =====================
# ===============================================

FROM base
COPY --from=builder /opencv/build /opencv/build

RUN \
    # install the build package
    cd /opencv/build && \
    ls -lh && \
    make install  --debug && \
    ldconfig -v && \
    rm -rf /opencv

RUN \
    # Call default command.
    ffmpeg -version && \
    #ldd `which ffmpeg` && \
    python --version && \
    pip list && \
    python -c "import cv2 ; print(cv2.__version__)"
