# ============================================
# CrossCell Benchmark - Base Image
# ============================================
# 最小化基础镜像：系统依赖 + R + Python + HDF5 + CrossCell
# R/Python 工具包在服务器上手动安装测试后再固化到镜像中
# ============================================

FROM ubuntu:22.04

LABEL maintainer="CrossCell Contributors"
LABEL description="CrossCell benchmark base environment"

ENV DEBIAN_FRONTEND=noninteractive

# ============================================
# 1. 系统依赖（全量，覆盖所有 R 包编译需求）
# ============================================
RUN apt-get update && apt-get install -y \
    build-essential \
    pkg-config \
    cmake \
    git \
    curl \
    wget \
    vim \
    ca-certificates \
    software-properties-common \
    dirmngr \
    gnupg \
    libhdf5-dev \
    hdf5-tools \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libglpk-dev \
    libopenblas-dev \
    zlib1g-dev \
    libbz2-dev \
    liblzma-dev \
    libwebp-dev \
    libgdal-dev \
    libgeos-dev \
    libproj-dev \
    libudunits2-dev \
    libgit2-dev \
    libfftw3-dev \
    python3 \
    python3-pip \
    python3-dev \
    python3-venv \
    time \
    procps \
    && rm -rf /var/lib/apt/lists/*

# ============================================
# 2. HDF5 1.14.6
# ============================================
RUN cd /tmp && \
    wget -q https://github.com/HDFGroup/hdf5/archive/refs/tags/hdf5_1.14.6.tar.gz -O hdf5-1.14.6.tar.gz && \
    tar xzf hdf5-1.14.6.tar.gz && \
    cd hdf5-hdf5_1.14.6 && \
    mkdir build && cd build && \
    cmake .. \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        -DCMAKE_BUILD_TYPE=Release \
        -DHDF5_BUILD_CPP_LIB=OFF \
        -DHDF5_BUILD_FORTRAN=OFF \
        -DHDF5_BUILD_JAVA=OFF \
        -DHDF5_BUILD_TOOLS=ON \
        -DHDF5_ENABLE_Z_LIB_SUPPORT=ON \
        -DBUILD_SHARED_LIBS=ON \
        -DBUILD_STATIC_LIBS=OFF \
        -DBUILD_TESTING=OFF && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    cd /tmp && rm -rf hdf5-hdf5_1.14.6 hdf5-1.14.6.tar.gz

# ============================================
# 3. R 4.5+
# ============================================
RUN wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | gpg --dearmor -o /usr/share/keyrings/r-project.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/r-project.gpg] https://cloud.r-project.org/bin/linux/ubuntu jammy-cran40/" > /etc/apt/sources.list.d/r-project.list && \
    apt-get update && \
    apt-get install -y r-base r-base-dev && \
    rm -rf /var/lib/apt/lists/*

# ============================================
# 4. 配置 R CRAN 镜像
# ============================================
RUN mkdir -p /usr/lib/R/etc && \
    echo 'options( \
  repos = c(CRAN = "https://cloud.r-project.org"), \
  download.file.method = "libcurl", \
  download.file.extra = "--retry 5 --retry-delay 3", \
  Ncpus = 4 \
)' >> /usr/lib/R/etc/Rprofile.site

# ============================================
# 5. Python 3.12 + venv
# ============================================
RUN add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y python3.12 python3.12-venv python3.12-dev && \
    rm -rf /var/lib/apt/lists/*

RUN python3.12 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir \
    numpy scipy pandas "anndata==0.12.3" scanpy h5py igraph leidenalg \
    jupyter notebook matplotlib

# ============================================
# 6. CrossCell 二进制
# ============================================
COPY benchmark/crosscell /usr/local/bin/crosscell
RUN chmod +x /usr/local/bin/crosscell

# ============================================
# 7. 目录和环境变量
# ============================================
RUN mkdir -p /benchmark/scripts /benchmark/data /benchmark/results

ENV BENCHMARK_DATA_DIR=/benchmark/data
ENV BENCHMARK_RESULTS_DIR=/benchmark/results
ENV R_LIBS_USER=/usr/local/lib/R/site-library

# ============================================
# 8. 验证基础环境
# ============================================
RUN echo "=== Verifying base environment ===" && \
    crosscell --version && \
    R --version | head -1 && \
    python3 --version && \
    python3 -c "import anndata; print('anndata', anndata.__version__)" && \
    jupyter --version | head -3 && \
    echo "=== Base environment OK ==="

WORKDIR /benchmark

CMD ["/bin/bash"]
