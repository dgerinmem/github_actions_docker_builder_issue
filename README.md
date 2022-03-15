# github_actions_docker_builder_issue

here is the action 
```bash
name: ......

on:
  push:
    branches: ['dev/dgerinmem']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME_BASE: ${{ github.repository }}/upmem_dpu

jobs:
  build-torch:
    runs-on: torch_cuda102_ubuntu_1804
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Log in to the Container registry
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME_BASE }}/ubuntu18

      - name: Build and push Docker image
        uses: docker/build-push-action@ad44023a93711e3deb337508980b4b5e9bcdc5dc
        with:
          context: torch/cuda102_ubuntu_1804/
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

the Dockerfile 

```bash
FROM nvidia/cuda:10.2-base-ubuntu18.04
ENV TORCH_VERSION v1.8.1

## Install essentials
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    sudo \
    git \
    bzip2 \
    libx11-6 \
    build-essential \
 && rm -rf /var/lib/apt/lists/*

## Install Anaconda
RUN curl -L https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh -o Anaconda3-2021.11-Linux-x86_64.sh
# RUN curl -L https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh -o Anaconda3-2020.11-Linux-x86_64.sh
# RUN curl -L https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh -o Anaconda3-2019.10-Linux-x86_64.sh
RUN chmod +x Anaconda3-2021.11-Linux-x86_64.sh
RUN ./Anaconda3-2021.11-Linux-x86_64.sh  -b; exit 0
RUN . /root/anaconda3/bin/activate && conda install astunparse numpy ninja pyyaml mkl mkl-include setuptools cmake cffi typing_extensions future six requests dataclasses --yes
RUN . /root/anaconda3/bin/activate && conda install -c pytorch magma-cuda110

## Install Pytorch
RUN . /root/anaconda3/bin/activate &&  git clone --recursive https://github.com/pytorch/pytorch
RUN cd pytorch && git checkout ${TORCH_VERSION}
RUN cd pytorch &&  git submodule sync
RUN cd pytorch &&  git submodule update --init --recursive --jobs 1
RUN cd pytorch &&   . /root/anaconda3/bin/activate && conda install -c anaconda conda-build
RUN cd pytorch &&  . /root/anaconda3/bin/activate && CMAKE_PREFIX_PATH=/root/anaconda3/bin python setup.py install
```


