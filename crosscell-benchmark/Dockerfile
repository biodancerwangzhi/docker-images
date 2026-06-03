# ============================================
# CrossCell Performance Benchmark Environment
# ============================================
# 用于对比 CrossCell 与其他工具的性能
# 包含: CrossCell, Zellkonverter, anndataR, convert2anndata, easySCF
# ============================================
# 编译流程：
#   1. 在 dev 容器中编译: docker-compose run --rm dev cargo build --release
#   2. 复制二进制: cp target/release/crosscell benchmark/
#   3. 构建本镜像: docker-compose -f benchmark/docker-compose.benchmark.yml build
# ============================================

FROM ubuntu:22.04

LABEL maintainer="CrossCell Contributors"
LABEL description="Performance benchmark environment for single-cell data conversion tools"

ENV DEBIAN_FRONTEND=noninteractive

# ============================================
# 1. 系统依赖（含 Seurat 编译所需的全部库）
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
    libuv1-dev \
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
# 5. R 基础包 + Bioconductor
# ============================================
RUN R -e "install.packages('BiocManager')"

RUN R -e "BiocManager::install(c( \
    'SingleCellExperiment', \
    'SummarizedExperiment', \
    'Biobase', \
    'S4Vectors', \
    'DelayedArray', \
    'rhdf5' \
), ask = FALSE, update = FALSE)"

RUN R -e "install.packages(c( \
    'Matrix', \
    'arrow', \
    'hdf5r', \
    'devtools', \
    'remotes', \
    'jsonlite', \
    'R6', \
    'cli', \
    'rlang' \
))"

# ============================================
# 6. Seurat V5
# ============================================
RUN R -e "install.packages('Seurat', dependencies = TRUE)" && \
    R -e "if (!requireNamespace('Seurat', quietly=TRUE)) quit(status=1, save='no')" && \
    R -e "library(Seurat); cat('Seurat', as.character(packageVersion('Seurat')), 'OK\n')"

# ============================================
# 7. Python 3.12 + venv
# ============================================
RUN add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y python3.12 python3.12-venv python3.12-dev && \
    rm -rf /var/lib/apt/lists/*

RUN python3.12 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir \
    numpy scipy pandas "anndata==0.12.3" scanpy h5py igraph leidenalg

# ============================================
# 8. convert2anndata（重试两次）
# ============================================
RUN R -e "library(remotes); install_github('settylab/convert2anndata', upgrade = 'never')" || \
    R -e "library(remotes); install_github('settylab/convert2anndata', upgrade = 'never')" || \
    echo "WARNING: convert2anndata install failed after 2 attempts"

# ============================================
# 9. easySCF (Python + R)
# ============================================
RUN pip install --no-cache-dir easySCFpy || echo "WARNING: easySCFpy install failed"

RUN R -e "library(remotes); install_url('https://github.com/xleizi/easySCF/archive/refs/heads/main.tar.gz', subdir='r')" || \
    echo "WARNING: easySCFr install failed"

# ============================================
# 10. Zellkonverter + anndataR (Bioconductor)
# ============================================
RUN R -e "BiocManager::install(c('zellkonverter', 'HDF5Array'), ask = FALSE, update = FALSE)" && \
    R -e "library(zellkonverter); cat('zellkonverter', as.character(packageVersion('zellkonverter')), 'OK\n')"

RUN R -e "BiocManager::install('anndataR', ask = FALSE, update = FALSE)" && \
    R -e "library(anndataR); cat('anndataR', as.character(packageVersion('anndataR')), 'OK\n')"

# ============================================
# 11. CrossCell 二进制
# ============================================
COPY benchmark/crosscell /usr/local/bin/crosscell
RUN chmod +x /usr/local/bin/crosscell

# ============================================
# 12. 目录和环境变量
# ============================================
RUN mkdir -p /benchmark/scripts /benchmark/data /benchmark/results

ENV BENCHMARK_DATA_DIR=/benchmark/data
ENV BENCHMARK_RESULTS_DIR=/benchmark/results
ENV R_LIBS_USER=/usr/local/lib/R/site-library
ENV BASILISK_USE_SYSTEM_DIR=1

# ============================================
# 13. 配置 basilisk (zellkonverter)
# ============================================
RUN PKGDIR=$(Rscript --slave -e "cat(system.file(package='zellkonverter'))") && \
    mkdir -p "${PKGDIR}/basilisk" && \
    ln -sf /opt/venv "${PKGDIR}/basilisk/zellkonverterAnnDataEnv-0.12.3" && \
    echo "Created symlink at ${PKGDIR}/basilisk/zellkonverterAnnDataEnv-0.12.3 -> /opt/venv"

# ============================================
# 14. anndata nullable strings patch
# ============================================
RUN ANNDATA_INIT=$(python3 -c "import anndata; print(anndata.__file__)") && \
    echo '' >> "${ANNDATA_INIT}" && \
    echo '# Auto-enable nullable strings writing (patched by CrossCell benchmark)' >> "${ANNDATA_INIT}" && \
    echo 'try:' >> "${ANNDATA_INIT}" && \
    echo '    settings.allow_write_nullable_strings = True' >> "${ANNDATA_INIT}" && \
    echo 'except: pass' >> "${ANNDATA_INIT}" && \
    python3 -c "import anndata; print('allow_write_nullable_strings:', anndata.settings.allow_write_nullable_strings)"

# ============================================
# 15. Jupyter + IRkernel
# ============================================
RUN pip install --no-cache-dir jupyter notebook matplotlib

RUN R -e "install.packages('IRkernel'); IRkernel::installspec(user = FALSE)"

# ============================================
# 16. 验证关键安装
# ============================================
RUN echo "=== Verifying CRITICAL installations ===" && \
    crosscell --version && \
    R -e "library(Seurat); cat('✅ Seurat', as.character(packageVersion('Seurat')), '\n')" && \
    R -e "library(zellkonverter); cat('✅ zellkonverter', as.character(packageVersion('zellkonverter')), '\n')" && \
    R -e "library(anndataR); cat('✅ anndataR', as.character(packageVersion('anndataR')), '\n')" && \
    R -e "library(rhdf5); cat('✅ rhdf5', as.character(packageVersion('rhdf5')), '\n')" && \
    jupyter kernelspec list && \
    echo "=== Critical packages OK ===" && \
    R -e "tryCatch(library(convert2anndata), error = function(e) cat('⚠ convert2anndata not available\n'))" && \
    R -e "tryCatch(library(easySCFr), error = function(e) cat('⚠ easySCFr not available\n'))" && \
    echo "=== All verifications complete ==="

WORKDIR /benchmark

CMD ["/bin/bash"]
