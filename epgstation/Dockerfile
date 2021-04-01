FROM nvidia/cuda:11.2.2-devel-ubuntu20.04

EXPOSE 8888
ENV DEBIAN_FRONTEND=noninteractive
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES video,compute,utility

ENV DEV="make gcc git g++ automake curl wget autoconf build-essential libass-dev libfreetype6-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev mercurial cmake clang llvm gcc-8 g++-8"

ARG FFMPEG_VER="4.3.2"

RUN set -xe && \
    apt-get update && \
    apt-get -y install $DEV && \
    apt-get -y install nasm yasm libx264-dev libmp3lame-dev libopus-dev libvpx-dev && \
    apt-get -y install libx265-dev libnuma-dev && \
    apt-get -y install libasound2 libass9 libvdpau1 libva-x11-2 libva-drm2 libxcb-shm0 libxcb-xfixes0 libxcb-shape0 libvorbisenc2 libtheora0 && \
    apt-get -y install npm python python3 libavdevice-dev libfdk-aac-dev qtbase5-dev && \
    mkdir -p /tmp/ffmpeg_sources

# nvidias codec API インストール

WORKDIR /tmp/ffmpeg_sources
RUN set -xe && \
    git -C nv-codec-headers pull 2> /dev/null || \
        git clone https://github.com/FFmpeg/nv-codec-headers && \
    cd nv-codec-headers && \
    make -j$(nproc) && \
    make install

WORKDIR /tmp/ffmpeg_sources
RUN set -xe && \
    export PATH=/usr/local/cuda/bin:$PATH &&\
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 10 && \
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 10 && \   
    wget -O /tmp/ffmpeg_sources/ffmpeg.tar.bz2 \
         https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VER}.tar.bz2 && \
    mkdir -p /tmp/ffmpeg_sources/ffmpeg && \
    tar jxvf /tmp/ffmpeg_sources/ffmpeg.tar.bz2 \
         -C /tmp/ffmpeg_sources/ffmpeg --strip-components 1 && \
    cd /tmp/ffmpeg_sources/ffmpeg && \
    ./configure \
    	--prefix=/usr/local \
        --disable-shared \
        --pkg-config-flags="--static" \
	--enable-gpl \
        --enable-libass \
	--enable-libfreetype \
	--enable-libmp3lame \
	--enable-libopus \
        --enable-libtheora \
	--enable-libvorbis \
	--enable-libvpx \
	--enable-libx264 \
	--enable-libx265 \
	--enable-nonfree \
	--disable-debug \
      	--disable-doc \
	--enable-libfdk-aac \
	--enable-static \
	--enable-cuda \
	--enable-cuvid \
	--enable-nvenc \
        --enable-nvdec \
	--enable-libnpp \
        --enable-ffnvcodec \
        --enable-cuda-nvcc \
        --enable-cuda-llvm \
	--extra-libs="-lpthread -lm" \
	--extra-cflags=-I/usr/local/cuda/include \
	--extra-ldflags=-L/usr/local/cuda/lib64 \
        --nvccflags="-gencode arch=compute_35,code=sm_35 -O2" && \
    make -j$(nproc) && \
    make install

# install EPGStation
RUN cd /usr/local/ && \
    git clone https://github.com/l3tnun/EPGStation.git && \
    cd /usr/local/EPGStation && \
    rm -rf config data thumbnail logs recorded && \
    ln -s /app/config config && \
    ln -s /app/data data && \
    ln -s /app/thumbnail thumbnail && \
    ln -s /app/logs logs && \
    ln -s /app/recorded recorded && \
    npm run all-install && \
    npm run build && \
\
# 不要なパッケージを削除
\
    apt-get autoremove -y && \
    apt-get clean&& \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/ffmpeg_sources

WORKDIR /usr/local/EPGStation

ENTRYPOINT npm start