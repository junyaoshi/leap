# We need the CUDA base dockerfile to enable GPU rendering
# on hosts with GPUs.
# The image below is a pinned version of nvidia/cuda:9.1-cudnn7-devel-ubuntu16.04 (from Jan 2018)
# If updating the base image, be sure to test on GPU since it has broken in the past.
#FROM nvidia/cuda:8.0-cudnn6-devel-ubuntu16.04
FROM nvidia/cudagl:11.4.0-runtime-ubuntu20.04

RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    curl \
    git \
    libgl1-mesa-dev \
    libgl1-mesa-glx \
    libglew-dev \
    libosmesa6-dev \
    software-properties-common \
    net-tools \
    unzip \
    vim \
    virtualenv \
    wget \
    xpra \
    xserver-xorg-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN DEBIAN_FRONTEND=noninteractive add-apt-repository --yes ppa:deadsnakes/ppa && apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install --yes python3.7-dev python3.7 python3-pip
RUN virtualenv --python=python3.7 env

#RUN rm /usr/bin/python
RUN rm /usr/bin/pip
RUN ln -s /env/bin/python3.7 /usr/bin/python
RUN ln -s /env/bin/pip3.7 /usr/bin/pip
RUN ln -s /env/bin/pytest /usr/bin/pytest

RUN curl -o /usr/local/bin/patchelf https://s3-us-west-2.amazonaws.com/openai-sci-artifacts/manual-builds/patchelf_0.9_amd64.elf \
    && chmod +x /usr/local/bin/patchelf

ENV LANG C.UTF-8

RUN mkdir -p /root/.mujoco \
    && wget https://www.roboti.us/download/mjpro150_linux.zip -O mujoco.zip \
    && unzip mujoco.zip -d /root/.mujoco \
    && rm mujoco.zip
COPY ./mjkey.txt /root/.mujoco/
ENV LD_LIBRARY_PATH /root/.mujoco/mjpro150/bin:${LD_LIBRARY_PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

COPY vendor/Xdummy /usr/local/bin/Xdummy
RUN chmod +x /usr/local/bin/Xdummy

# Workaround for https://bugs.launchpad.net/ubuntu/+source/nvidia-graphics-drivers-375/+bug/1674677
COPY ./vendor/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

WORKDIR /mujoco_py
# Copy over just requirements.txt at first. That way, the Docker cache doesn't
# expire until we actually change the requirements.
COPY ./requirements.txt /mujoco_py/
COPY ./requirements.dev.txt /mujoco_py/

RUN python -m pip install --upgrade pip
RUN python -m pip uninstall -y numpy
RUN python -m pip install -r requirements.txt
RUN python -m pip install -r requirements.dev.txt
RUN python -m pip install lockfile

#RUN python -m pip install cloudpickle==0.5.2
RUN python -m pip install cached-property
# For atari-py
RUN apt-get install -y zlib1g-dev swig cmake

#RUN python -m pip install mujoco_py==1.50.1.56
RUN python -m pip install -U 'mujoco-py<1.50.2,>=1.50.1'
RUN python -c 'import mujoco_py'


RUN python -m pip install gym[all]==0.15.4
RUN python -m pip install gitpython
RUN python -m pip install gtimer
RUN python -m pip install pygame
RUN python -m pip install awscli
RUN python -m pip install boto3
RUN python -m pip install dominate
RUN python -m pip install ray
RUN python -m pip install path.py

#RUN python -m pip install http://download.pytorch.org/whl/cu80/torch-0.3.0.post4-cp35-cp35m-linux_x86_64.whl
#RUN python -m pip install http://download.pytorch.org/whl/cu90/torch-0.3.0.post4-cp35-cp35m-linux_x86_64.whl
RUN python -m pip install torch
RUN python -m pip install joblib

# If you're curious what the DEBIAN_FRONTEND is all about:
# https://askubuntu.com/questions/506158/unable-to-initialize-frontend-dialog-when-using-ssh
#RUN apt-get purge nvidia*
#RUN add-apt-repository --yes ppa:graphics-drivers && apt-get update

RUN apt-get update && apt-get install -y ffmpeg

# For some reason this works despite an error showing up...
#RUN DEBIAN_FRONTEND=noninteractive apt-get -qy install nvidia-384; exit 0
#ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/lib/nvidia-384

RUN mkdir -p /usr/lib/nvidia-000
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/lib/nvidia-000

RUN mkdir /root/code
WORKDIR /root/code

# trigger the mujoco_py cythonization
RUN python -c 'import mujoco_py'

RUN python -m pip install opencv-python
RUN python -m pip install torchvision
RUN python -m pip install sk-video
RUN python -m pip install matplotlib
RUN python -m pip install numpy-stl
RUN python -m pip install pyquaternion

RUN python -m pip uninstall numpy -y
RUN python -m pip install numpy

RUN python -m pip install gin-config
RUN python -m pip install tensorflow-gpu
RUN python -m pip install tf-agents
RUN python -m pip install tensorflow-probability
