# Use NVIDIA base image
FROM nvidia/cuda:11.2.2-runtime-ubuntu20.04

ARG DEBIAN_FRONTEND=noninteractive

WORKDIR /yolo_orbslam3

# Install necessary dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    wget \
    unzip \
    software-properties-common \
    libgtk2.0-dev \
    pkg-config \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    python-dev \
    python-numpy \
    libtbb2 \
    libtbb-dev \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libdc1394-22-dev \
    libopenjp2-7-dev \
    libglew-dev \
    libboost-dev \
    libboost-thread-dev \
    libboost-filesystem-dev \
    libpython2.7-dev \
    python3-dev \
    python3-numpy \
    libboost-program-options-dev \
    libboost-regex-dev \
    libboost-system-dev \
    libboost-test-dev \
    libboost-thread-dev \
    ffmpeg \
    libavutil-dev \
    libavfilter-dev \
    libavdevice-dev \
    libavresample-dev \
    libglvnd-dev \
    gdb \
    xvfb \
    libgl1-mesa-glx \
    libglib2.0-0 \
    freeglut3-dev \
    mesa-utils \
    ca-certificates \
    libssl-dev \
    python3-pip

# Add repository for GCC 11 and install it
RUN add-apt-repository ppa:ubuntu-toolchain-r/test && \
    apt-get update && \
    apt-get install -y \
    gcc-11 \
    g++-11

# Set GCC 11 as default
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100 && \
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 100

# Download and install OpenCV binary
RUN git clone https://github.com/opencv/opencv.git && \
    cd opencv/ && \
    git checkout 4.2.0 && \
    mkdir build && \
    cd build && \
    cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..  && \
    make install

# Install Eigen
RUN git clone https://gitlab.com/libeigen/eigen.git && \
    cd eigen/ && \
    git checkout 3.3.9 && \
    mkdir build && cd build && \
    cmake .. && \
    make -j$(nproc) && \
    make install

# Install Pangolin
RUN git clone https://github.com/stevenlovegrove/Pangolin.git && \
    cd Pangolin/ && \
    git checkout v0.6 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_C_COMPILER=gcc-11 -DCMAKE_CXX_COMPILER=g++-11 -DCMAKE_CXX_FLAGS="-Wno-catch-value -Wno-deprecated-declarations" -DOpenGL_GL_PREFERENCE=LEGACY && \
    make -j$(nproc) && \
    make install

# Install libtorch
RUN wget https://download.pytorch.org/libtorch/cpu/libtorch-cxx11-abi-shared-with-deps-1.11.0%2Bcpu.zip && \
    unzip libtorch-cxx11-abi-shared-with-deps-1.11.0+cpu.zip && \
    mv libtorch /yolo_orbslam3/Thirdparty

RUN pip3 install torch torchvision

# Set the Torch_DIR environment variable
ENV Torch_DIR=/yolo_orbslam3/Thirdparty/libtorch/share/cmake/Torch
ENV CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}:/yolo_orbslam3/Thirdparty/libtorch

# Copy repository into container
COPY ./YOLO_ORB_SLAM3 /yolo_orbslam3

# Build the project
RUN chmod +x build.sh && ./build.sh

# Set library paths for NVIDIA drivers
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH

# Ensure the shared library cache is updated
RUN ldconfig

# Set environment variables and execute your application
CMD ["bash", "-c", "ldconfig && Xvfb :1 -screen 0 1024x768x24 & export DISPLAY=:1 && ./Examples/RGB-D/rgbd_tum Vocabulary/ORBvoc.txt Examples/RGB-D/TUM3.yaml /yolo_orbslam3/TUM_dataset/ /yolo_orbslam3/TUM_dataset/associate.txt"]
