ARG UBUNTU_VER=21.04

FROM ubuntu:${UBUNTU_VER} AS dpdkbuilder
ARG DEBIAN_FRONTEND=noninteractive

ARG DPDK_VER=20.11
ARG PREFIX_DIR=/usr/local

#wget https://fast.dpdk.org/rel/dpdk-$DPDK_VER.tar.xz
COPY ./dpdk-${DPDK_VER}.tar.xz dpdk-${DPDK_VER}.tar.xz

RUN sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get -y install python3 \
               meson libbpf-dev libpcap-dev \
              ninja-build \
              libnuma-dev lua5.4-dev \
               wget pkg-config \
               build-essential linux-headers-generic && \
#    wget https://fast.dpdk.org/rel/dpdk-$DPDK_VER.tar.xz && \
    tar -xf dpdk-${DPDK_VER}.tar.xz && \
    cd dpdk-$DPDK_VER && \
    meson build -Dtests=false -Dexamples=all -Dprefix=${PREFIX_DIR}/dpdk && \
    ninja -C build -j4 install && \
    ldconfig ${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/

FROM dpdkbuilder AS pktgenbuilder

ARG PREFIX_DIR=/usr/local
ARG PKGGEN_VER=20.11.3
COPY ./pktgen-${PKGGEN_VER}.tar.xz pktgen-${PKGGEN_VER}.tar.xz
ENV PKG_CONFIG_PATH "${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/pkgconfig"
RUN cd / && \
#    wget https://git.dpdk.org/apps/pktgen-dpdk/snapshot/pktgen-${PKGGEN_VER}.tar.xz && \
    tar -xf pktgen-${PKGGEN_VER}.tar.xz && \
    cd pktgen-${PKGGEN_VER} && \
    CFLAGS=-I${PREFIX_DIR}/dpdk/include meson build -Dprefix=${PREFIX_DIR}/dpdk/ -Denable_lua=true && \
    ninja -C build install -j4

# Use same alpine as the base for the runtime image
FROM ubuntu:${UBUNTU_VER}
ARG DEBIAN_FRONTEND=noninteractive

#LABEL RUN docker run -it --privileged -v /sys/bus/pci/devices:/sys/bus/pci/devices \
#        -v /sys/kernel/mm/hugepages:/sys/kernel/mm/hugepages \
#        -v /sys/devices/system/node:/sys/devices/system/node \
#        -v /dev:/dev --name NAME -e NAME=NAME -e IMAGE=IMAGE IMAGE"

MAINTAINER calvin <c6supper@hotmail.com>

ARG PREFIX_DIR=/usr/local

# Copy build artifacts into this stage
COPY --from=pktgenbuilder ${PREFIX_DIR}/dpdk ${PREFIX_DIR}/dpdk
ENV PATH="${PREFIX_DIR}/dpdk/bin:${PATH}"
RUN sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get -y install libnuma1 libbpf0 libpcap0.8 lua5.4 && \
    ldconfig ${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/ && \
    echo "over"

CMD ["/usr/bin/bash"]
