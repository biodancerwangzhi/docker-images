# ============================================
# CrossCell Development Container
# GitHub Actions 自动构建 -> DockerHub: biodancer/crosscell-dev
# 
# 触发构建: git tag crosscell-dev/v1.0.0 && git push --tags
# ============================================
FROM satijalab/seurat:5.0.0

LABEL maintainer="biodancer"
LABEL description="CrossCell: Single-cell data format conversion toolkit (AnnData <-> Seurat)"
LABEL version="1.0.0"

ENV DEBIAN_FRONTEND=noninteractive

# ============================================
# 安装基础系统工具
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
    libclang-dev \
    llvm-dev \
    zlib1g-dev \
    libbz2-dev \
    liblzma-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    hdf5-tools \
    libfontconfig1-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    libwebp-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libglpk-dev \
    libopenblas-dev \
    python3 \
    python3-pip \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# ============================================
# 安装 CMake >= 3.26（HDF5 静态编译需要）
# ============================================
RUN wget -q https://github.com/Kitware/CMake/releases/download/v3.31.4/cmake-3.31.4-linux-x86_64.tar.gz -O /tmp/cmake.tar.gz && \
    tar xzf /tmp/cmake.tar.gz -C /opt/ && \
    rm /tmp/cmake.tar.gz
ENV PATH=/opt/cmake-3.31.4-linux-x86_64/bin:$PATH

# ============================================
# 安装 Rust
# ============================================
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable
ENV PATH=/root/.cargo/bin:$PATH

RUN rustup component add rustfmt clippy && \
    cargo install cargo-watch

# ============================================
# 安装 Python 科学计算库
# ============================================
RUN pip install --no-cache-dir \
    numpy \
    scipy \
    pandas \
    anndata \
    scanpy \
    igraph \
    leidenalg \
    scvelo \
    diopy \
    easySCFpy \
    pytest \
    hypothesis

# ============================================
# 安装 R 包（easySCF, dior）
# ============================================
RUN R -e "install.packages('rlang', repos='https://cloud.r-project.org'); \
          install.packages('remotes', repos='https://cloud.r-project.org'); \
          options(download.file.method = 'libcurl'); \
          remotes::install_github('xleizi/easySCF/r', upgrade = 'never', force = TRUE); \
          remotes::install_github('JiekaiLab/dior', upgrade = 'never', force = TRUE)"

# ============================================
# 环境变量
# ============================================
ENV RUST_BACKTRACE=1
ENV CARGO_INCREMENTAL=1
ENV PYTHONPATH=/workspace/python

WORKDIR /workspace

CMD ["/bin/bash"]
