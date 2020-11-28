ARG UBUNTU_VER=21.04

FROM ubuntu:${UBUNTU_VER} AS builder

ARG DPDK_VER=20.11
ARG PREFIX_DIR=/usr/local
ARG PKGGEN_VER=20.11.3

#wget https://fast.dpdk.org/rel/dpdk-$DPDK_VER.tar.xz
#COPY ./dpdk-${DPDK_VER}.tar.xz dpdk-${DPDK_VER}.tar.xz

RUN sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get -y install python3 \
               meson libbpf-dev libpcap-dev \
               ninja-build \
               libnuma-dev lua5.4-dev \
               wget pkg-config \
               build-essential && \
    wget https://fast.dpdk.org/rel/dpdk-$DPDK_VER.tar.xz && \
    tar -xf dpdk-${DPDK_VER}.tar.xz && \
    cd dpdk-$DPDK_VER && \
    meson build -Dtests=false -Dexamples=all -Dprefix=${PREFIX_DIR}/dpdk && \
    ninja -C build -j4 install && \
    ldconfig ${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/ && \
    cd / && \
    wget https://git.dpdk.org/apps/pktgen-dpdk/snapshot/pktgen-${PKGGEN_VER}.tar.xz && \
    tar -xf pktgen-${PKGGEN_VER}.tar.xz && \
    cd pktgen-${PKGGEN_VER} && \
    export PKG_CONFIG_PATH=\$PKG_CONFIG_PATH:${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/pkgconfig && \
    make buildlua -j4 && \
    make install

# Use same alpine as the base for the runtime image
FROM ubuntu:${UBUNTU_VER}

LABEL RUN docker run -it --privileged -v /sys/bus/pci/devices:/sys/bus/pci/devices \
        -v /sys/kernel/mm/hugepages:/sys/kernel/mm/hugepages \
        -v /sys/devices/system/node:/sys/devices/system/node \
        -v /dev:/dev --name NAME -e NAME=NAME -e IMAGE=IMAGE IMAGE"

MAINTAINER calvin <c6supper@hotmail.com>

ARG PREFIX_DIR=/usr/local

# Copy build artifacts into this stage
COPY --from=builder ${PREFIX_DIR}/dpdk ${PREFIX_DIR}/dpdk
COPY --from=builder /pktgen-${PKGGEN_VER}/usr/local/* ${PREFIX_DIR}/

RUN sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get -y install python3 libnuma libbpf libpcap lua5.4 && \
    ldconfig ${PREFIX_DIR}/dpdk/lib/x86_64-linux-gnu/ && \
    ldconfig ${PREFIX_DIR}/pktgen/lib/x86_64-linux-gnu/ && \
    echo 'export \$PATH=\$PATH:${PREFIX_DIR}/dpdk/bin' > /tmp/bashrc && \
    mkdir /mnt/huge && \
    mount -t hugetlbfs nodev /mnt/huge && \
    echo "over"

CMD ["/usr/bin/bash"]
