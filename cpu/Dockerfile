# https://www.learnopencv.com/install-opencv3-on-ubuntu/
# https://www.osradar.com/how-to-install-opencv-on-ubuntu-20-04/

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

# ============== BASE IMAGE =====================
# ===============================================

RUN \
    # add sources for older pythons
    apt-get update --fix-missing && \
    apt-get install -y --no-install-recommends software-properties-common && \
    if [[ "$PYTHON_VERSION" =~ ^(2.7|3.6|3.7|3.8|3.9)$ ]]; then \
        add-apt-repository ppa:deadsnakes/ppa ; \
    fi && \
    apt-add-repository universe && \
    apt-get -y update -qq --fix-missing && \
    apt-get -y install --no-install-recommends \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-dev \
        $( [ ${PYTHON_VERSION%%.*} -ge 3 ] && echo "python${PYTHON_VERSION}-distutils" ) \
        wget && \
    rm -rf /var/lib/apt/lists/* && \

    # Set the default python and install PIP packages
    update-alternatives --install /usr/bin/python${PYTHON_VERSION%%.*} python${PYTHON_VERSION%%.*} /usr/bin/python${PYTHON_VERSION} 1 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 1

ENV PYTHONPATH="/usr/lib/python${PYTHON_VERSION}/site-packages:/usr/local/lib/python${PYTHON_VERSION}/site-packages"

RUN \
    # sysctl -w net.ipv4.ip_forward=1 ; \
    if [[ "$PYTHON_VERSION" =~ ^(2.7|3.6)$ ]]; then \
        PIP_URL="https://bootstrap.pypa.io/pip/$PYTHON_VERSION" ; \
    else \
        PIP_URL="https://bootstrap.pypa.io" ; \
    fi && \
    wget -q "${PIP_URL}/get-pip.py" --no-check-certificate && \
    python get-pip.py && \
    rm get-pip.py



# ============ COMPILE OPENCV ===================
# ===============================================

FROM base AS builder

RUN \
    # Install all dependencies for OpenCV
    apt-get -y update -qq --fix-missing && \
    apt-get -y install --no-install-recommends \
        unzip \
        cmake \
        ffmpeg \
        libtbb2 \
        gfortran \
        apt-utils \
        pkg-config \
        checkinstall \
        qt5-default \
        build-essential \
        libopenblas-base \
        libopenblas-dev \
        liblapack-dev \
        libatlas-base-dev \
        #libgtk-3-dev \
        #libavcodec58 \
        libavcodec-dev \
        #libavformat58 \
        libavformat-dev \
        libavutil-dev \
        #libswscale5 \
        libswscale-dev \
        libjpeg8-dev \
        libpng-dev \
        libtiff5-dev \
        #libdc1394-22 \
        libdc1394-22-dev \
        libxine2-dev \
        libv4l-dev \
        libgstreamer1.0 \
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
        libxrender1 \

RUN \
    pip install numpy && \
# Install OpenCV
    wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip -O opencv.zip --progress=bar:force:noscroll --no-check-certificate && \
    unzip -q opencv.zip && \
    mv /opencv-${OPENCV_VERSION} /opencv && \
    rm opencv.zip && \
    wget https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip -O opencv_contrib.zip --progress=bar:force:noscroll --no-check-certificate && \
    unzip -q opencv_contrib.zip && \
    mv /opencv_contrib-${OPENCV_VERSION} /opencv_contrib && \
    rm opencv_contrib.zip && \

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
      #-D WITH_HPX=ON \
      -D ENABLE_PRECOMPILED_HEADERS=OFF \
      .. \
    && \

# Build, Test and Install
    cd /opencv/build && \
    make -j$(nproc)

# ============= FINAL IMAGE =====================
# ===============================================

FROM base
COPY --from=builder /opencv/build /tmp/opencv/build

RUN \
    cd /tmp/opencv/build && \
    make install && \
    ldconfig && \

# Call default command.
    ffmpeg -version && \
    #ldd `which ffmpeg` && \
    python --version && \
    python -c "import cv2 ; print(cv2.__version__)"
