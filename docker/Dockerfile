ARG CUDA_VERSION=12.4.1

FROM nvcr.io/nvidia/tritonserver:24.12-py3-min

ARG BUILD_TYPE=all
ENV DEBIAN_FRONTEND=noninteractive

RUN echo 'tzdata tzdata/Areas select America' | debconf-set-selections \
    && echo 'tzdata tzdata/Zones/America select Los_Angeles' | debconf-set-selections \
    && apt update -y \
    && apt install software-properties-common -y \
    && apt install python3 python3-pip -y \
    && apt install curl git sudo libibverbs-dev -y \
    && apt install rdma-core infiniband-diags openssh-server perftest -y \
    && python3 --version \
    && python3 -m pip --version \
    && rm -rf /var/lib/apt/lists/* \
    && apt clean

# For openbmb/MiniCPM models
RUN pip3 install datamodel_code_generator --break-system-packages

WORKDIR /sgl-workspace

ARG CUDA_VERSION
RUN python3 -m pip install --upgrade pip setuptools wheel html5lib six --break-system-packages --ignore-installed \
    && git clone --depth=1 https://github.com/sebin-maret/sglang.git \
    && if [ "$CUDA_VERSION" = "12.1.1" ]; then \
         export CUINDEX=121; \
       elif [ "$CUDA_VERSION" = "12.4.1" ]; then \
         export CUINDEX=124; \
       elif [ "$CUDA_VERSION" = "12.8.1" ]; then \
         export CUINDEX=124; \
       elif [ "$CUDA_VERSION" = "11.8.0" ]; then \
         export CUINDEX=118; \
         python3 -m pip install --no-cache-dir sgl-kernel -i https://docs.sglang.ai/whl/cu118 --break-system-packages; \
       else \
         echo "Unsupported CUDA version: $CUDA_VERSION" && exit 1; \
       fi \
    && if [ "$CUDA_VERSION" = "12.4.1" ]; then \
         python3 -m pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cu126 --break-system-packages; \
       else \
         python3 -m pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cu${CUINDEX} --break-system-packages; \
       fi \
    && cd sglang \
    && python3 -m pip --no-cache-dir install -e "python[${BUILD_TYPE}]" --break-system-packages \
    && if [ "$CUDA_VERSION" = "12.8.1" ]; then \
         python3 -m pip install nvidia-nccl-cu12==2.26.2.post1 --force-reinstall --no-deps --break-system-packages; \
       fi

ENV DEBIAN_FRONTEND=interactive
