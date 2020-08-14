# Docker: Python-OpenCV-FFmpeg(-CUDA)

Repository for clean Dockerfile containing [FFmpeg](https://www.ffmpeg.org/), [OpenCV4](https://opencv.org/) and [Python2/3](https://www.python.org/), based on [Ubuntu](https://www.ubuntu.com/) 20.04 LTS.


## Tags

* `:cpu` Python 2.x/3.x, OpenCV 4.4.0, FFmpeg
* `:gpu` Python 2.x/3.x, OpenCV 4.4.0, FFmpeg with CUDA 11.0 support


## Build

[![CircleCI](https://circleci.com/gh/Borda/docker_python-opencv-ffmpeg/tree/master.svg?style=svg)](https://circleci.com/gh/Borda/docker_python-opencv-ffmpeg/tree/master)
[![Docker Build Status](https://img.shields.io/docker/build/borda/docker_python-opencv-ffmpeg)](https://hub.docker.com/r/borda/docker_python-opencv-ffmpeg)
[![DockerHub Pulls](https://img.shields.io/docker/pulls/borda/docker_python-opencv-ffmpeg.svg)](https://hub.docker.com/r/borda/docker_python-opencv-ffmpeg)
[![Docker](https://img.shields.io/docker/automated/borda/docker_python-opencv-ffmpeg)](https://hub.docker.com/r/borda/docker_python-opencv-ffmpeg)
<!--
[![Build Status](https://travis-ci.org/Borda/docker_python-opencv-ffmpeg.svg?branch=master)](https://travis-ci.org/Borda/docker_python-opencv-ffmpeg)
-->


First you need to install docker on your local computer, see following [tutorial](https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository). Note, for running the docker properly you have be logged as superuser otherwise you will face many partial issues which sometimes does not make much sense.

You can build it on your own, note it takes lots of time, be prepared.
```bash
git clone <git-repository>
cd docker_python-opencv-ffmpeg
docker image build -t python-opencv-ffmpeg:py36 -f cpu/Dockerfile --build-arg PYTHON_VERSION=3.6 .
```
To build other versions, select different Dockerfile.
```bash
docker image list
docker run --rm -it python-opencv-ffmpeg:py36 bash
docker image rm python-opencv-ffmpeg:py36
```

Other option is using already build image from DockerHub which is significantly faster. it basically download the already build image.
``` bash
docker pull borda/docker_python-opencv-ffmpeg
```

**Cleaning**
In case you fail with some builds, you may need to clean your local storage.
```bash
docker image prune
```
or [Docker - How to cleanup (unused) resources](https://gist.github.com/bastman/5b57ddb3c11942094f8d0a97d461b430)
```bash
docker images | grep "none"
docker rmi $(docker images | grep "none" | awk '/ / { print $3 }')
```
or remove all - [Some way to clean up](https://forums.docker.com/t/some-way-to-clean-up-identify-contents-of-var-lib-docker-overlay/30604)
```bash
docker rm -vf $(docker ps -aq)
docker rmi -f $(docker images -aq)
docker volume prune -f
```


## Usage

Image has OpenCV4, Python2.7/3.6 and FFmpeg ready to use. Example:

``` bash
docker run --rm -it -v $PWD:/srv borda/docker_python-opencv-ffmpeg python
>>> import cv2; cv2.VideoCapture(0).read()
# truncated for transparency
(True, array([[[ 0, 43, 37], ...]], dtype=uint8))
```

Note, server usually doe not have webcam :)
