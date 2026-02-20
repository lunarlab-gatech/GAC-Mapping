FROM ros:melodic-ros-base

# ============= Dependencies for GAC-Mapping =================

# 1. System build tools and additional ROS packages (Prerequisite 1.1)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    git \
    wget \
    sudo \
    ros-melodic-cv-bridge \
    ros-melodic-tf \
    ros-melodic-message-filters \
    ros-melodic-image-transport \
    ros-melodic-rospy-tutorials \
    # Ceres Solver build dependencies (Prerequisite 1.2)
    libgoogle-glog-dev \
    libgflags-dev \
    libatlas-base-dev \
    libeigen3-dev \
    libsuitesparse-dev \
    && rm -rf /var/lib/apt/lists/*

# 2. Build and install Ceres Solver 2.0.0 (Prerequisite 1.2)
RUN git clone --branch 2.0.0 https://github.com/ceres-solver/ceres-solver.git /tmp/ceres && \
    mkdir -p /tmp/ceres/build && \
    cd /tmp/ceres/build && \
    cmake .. -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -rf /tmp/ceres

# 3. Install Mambaforge system-wide (Prerequisite 1.3 – Patch-NetVLAD)
#    Mambaforge ships mamba, which resolves envs far faster than conda.
#    Pin to the last release that supports Ubuntu 18.04 (GLIBC 2.27).
RUN wget -q https://github.com/conda-forge/miniforge/releases/download/23.1.0-1/Mambaforge-23.1.0-1-Linux-x86_64.sh \
         -O /tmp/mambaforge.sh && \
    bash /tmp/mambaforge.sh -b -p /opt/conda && \
    rm /tmp/mambaforge.sh && \
    # Make conda/mamba available to all users in interactive bash shells
    echo '. /opt/conda/etc/profile.d/conda.sh' >> /etc/bash.bashrc

# 4. Create the patchnetvlad conda environment (CPU build; swap pytorch-cpu →
#    pytorch-gpu and add cudatoolkit if running under nvidia-docker)
RUN /opt/conda/bin/mamba create -n patchnetvlad python=3.8 \
        numpy pytorch-cpu torchvision natsort tqdm opencv pillow \
        scikit-learn faiss matplotlib-base -c conda-forge -y && \
    /opt/conda/bin/conda run -n patchnetvlad \
        pip install --no-cache-dir pyyaml rospkg && \
    /opt/conda/bin/conda clean -afy

# 5. Install PCL (Point Cloud Library) required by gacm
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-melodic-pcl-ros \
    libpcl-dev \
    && rm -rf /var/lib/apt/lists/*

# Set user arguments (being username, echo $UID, and id -g)
ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=1000

# Create a non-root user with matching UID/GID
RUN groupadd --gid ${USER_GID} ${USERNAME} && \
    useradd --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME} && \
    usermod -aG sudo ${USERNAME} && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Add a custom .bashrc with warm colors and fixed hostname
RUN echo 'export PS1="\\[\\e[1;31m\\]\\u@gac_mapping_container\\[\\e[0m\\]:\\[\\e[1;33m\\]\\w\\[\\e[0m\\]\\$ "' > /home/${USERNAME}/.bashrc && \
    chown ${USERNAME}:${USERNAME} /home/${USERNAME}/.bashrc

# Set default user & shell
USER ${USERNAME}
WORKDIR /home/${USERNAME}
SHELL ["/bin/bash", "-c"]
RUN echo "export USER=${USERNAME}" >> ~/.bashrc