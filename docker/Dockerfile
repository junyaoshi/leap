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

RUN rm /usr/bin/python
RUN ln -s /env/bin/python3.7 /usr/bin/python
RUN ln -s /env/bin/pip3.7 /usr/bin/pip
RUN ln -s /env/bin/pytest /usr/bin/pytest

RUN curl -o /usr/local/bin/patchelf https://s3-us-west-2.amazonaws.com/openai-sci-artifacts/manual-builds/patchelf_0.9_amd64.elf \
    && chmod +x /usr/local/bin/patchelf

ENV LANG C.UTF-8

RUN mkdir -p /root/.mujoco \
    && wget https://roboti.us/download/mujoco200_linux.zip -O mujoco.zip \
    && unzip mujoco.zip -d /root/.mujoco \
    && rm mujoco.zip
COPY ./mjkey.txt /root/.mujoco/
ENV LD_LIBRARY_PATH /root/.mujoco/mujoco200/bin:${LD_LIBRARY_PATH}
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
RUN pip install -r requirements.txt
RUN pip install -r requirements.dev.txt
#RUN pip install cloudpickle==0.5.2
RUN pip install cached-property
# For atari-py
RUN apt-get install -y zlib1g-dev swig cmake
RUN pip install mujoco_py
RUN pip install gym[all]
RUN pip install gitpython
RUN pip install gtimer
RUN pip install pygame
RUN pip install awscli
RUN pip install boto3
RUN pip install dominate
RUN pip install ray
RUN pip install path.py

#RUN pip install http://download.pytorch.org/whl/cu80/torch-0.3.0.post4-cp35-cp35m-linux_x86_64.whl
#RUN pip install http://download.pytorch.org/whl/cu90/torch-0.3.0.post4-cp35-cp35m-linux_x86_64.whl
RUN pip install torch
RUN pip install joblib

# If you're curious what the DEBIAN_FRONTEND is all about:
# https://askubuntu.com/questions/506158/unable-to-initialize-frontend-dialog-when-using-ssh
#RUN apt-get purge nvidia*
#RUN add-apt-repository --yes ppa:graphics-drivers && apt-get update

RUN apt-get update && apt-get install -y libav-tools

# For some reason this works despite an error showing up...
#RUN DEBIAN_FRONTEND=noninteractive apt-get -qy install nvidia-384; exit 0
#ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/lib/nvidia-384

RUN mkdir -p /usr/lib/nvidia-000
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/lib/nvidia-000

RUN mkdir /root/code
WORKDIR /root/code

# trigger the mujoco_py cythonization
RUN python -c 'import mujoco_py'

RUN pip install opencv-python
RUN pip install torchvision
RUN pip install sk-video
RUN pip install matplotlib
RUN pip install numpy-stl
RUN pip install pyquaternion

RUN pip uninstall numpy -y
RUN pip install numpy

RUN pip install gin-config
RUN pip install tensorflow-gpu
RUN pip install tf-agents
RUN pip install tensorflow-probability
