ARG BUILD_FROM=intel/oneapi:2025.0.1-0-devel-ubuntu24.04
FROM ${BUILD_FROM}

ARG WHISPER_CPP_VERSION=1.7.4
ARG NEO_RUNTIME_VERSION=24.26.30049.6
ARG NEO_LZ_VERSION=1.3.30049.6
ARG IGC_VERSION=1.0.17193.4
ARG GMM_VERSION=22.3.20

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        netcat-traditional \
        wget \
        ocl-icd-libopencl1 \
        clinfo

RUN \
    apt-get remove -y intel-opencl-icd intel-level-zero-gpu \
        intel-igc-core intel-igc-opencl || true \
    && mkdir -p /tmp/neo && cd /tmp/neo \
    && base=https://github.com/intel/compute-runtime/releases/download/${NEO_RUNTIME_VERSION} \
    && igc=https://github.com/intel/intel-graphics-compiler/releases/download/igc-${IGC_VERSION} \
    && wget -q $igc/intel-igc-core_${IGC_VERSION}_amd64.deb \
    && wget -q $igc/intel-igc-opencl_${IGC_VERSION}_amd64.deb \
    && wget -q $base/intel-level-zero-gpu_${NEO_LZ_VERSION}_amd64.deb \
    && wget -q $base/intel-opencl-icd_${NEO_RUNTIME_VERSION}_amd64.deb \
    && wget -q $base/libigdgmm12_${GMM_VERSION}_amd64.deb \
    && (dpkg -i *.deb || apt-get install -f -y) \
    && cd / && rm -rf /tmp/neo /var/lib/apt/lists/*

RUN \
    git clone https://github.com/ggerganov/whisper.cpp.git whisper.cpp \
    && cd whisper.cpp \
    && git reset --hard v${WHISPER_CPP_VERSION} \
    && cmake -B build \
        -DGGML_SYCL=ON \
        -DGGML_SYCL_F16=ON \
        -DGGML_NATIVE=OFF \
        -DGGML_AVX512=OFF \
        -DGGML_AVX2=ON \
        -DGGML_FMA=ON \
        -DGGML_F16C=ON \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_COMPILER=icx \
        -DCMAKE_CXX_COMPILER=icpx \
    && cmake --build build -j --config Release

WORKDIR /whisper.cpp

