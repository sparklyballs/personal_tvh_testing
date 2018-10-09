FROM lsiobase/ubuntu:bionic as comskip_buildstage

# package versions
ARG FFMPEG="ffmpeg-4.0.2"

# environment settings
ARG DEBIAN_FRONTEND="noninteractive"

RUN \
 echo "**** install build packages ****" && \
 apt-get update && \
 apt-get install -y \
	autoconf \
	automake \
	build-essential \
	bzip2 \
	git \
	libargtable2-dev \
	libtool \
	pkg-config \
	wget \
	xz-utils \
	yasm

RUN \
 echo "**** fetch ffmpeg source ****" && \
 wget --quiet http://ffmpeg.org/releases/${FFMPEG}.tar.bz2 && \
 tar xf ${FFMPEG}.tar.bz2 -C /tmp

RUN \
 echo "**** compile ffmpeg ****" && \
 cd /tmp/${FFMPEG} && \
 ./configure \
	--disable-programs \
	--prefix=/tmp/comskipbuild/install && \
 make -j4 && \
 make install

RUN \
 echo "**** compile comskip ****" && \
 git clone https://github.com/erikkaashoek/Comskip.git /tmp/comskip && \
 cd /tmp/comskip && \
 ./autogen.sh && \
 PKG_CONFIG_PATH=/tmp/comskipbuild/install/lib/pkgconfig \
 ./configure \
	--bindir=/tmp/binaries/static/bin \
	--enable-static \
	--sysconfdir=/config/comskip && \
 make -j4 && \
 make install && \
 strip --strip-all /tmp/binaries/static/bin/comskip

FROM lsiobase/ubuntu:bionic as tvh_buildstage

# environment settings
ARG DEBIAN_FRONTEND="noninteractive"

RUN \
 echo "**** install build packages ****" && \
 apt-get update && \
 apt-get install -y \
	build-essential \
	bzip2 \
	cmake \
	gettext \
	git \
	libavahi-client-dev \
	libavcodec-dev \
	libavformat-dev \
	libavresample-dev \
	libavutil-dev \
	libdvbcsa-dev \
	libpcre2-dev \
	libssl-dev \
	libswscale-dev \
	liburiparser-dev \
	pkg-config \
	pngquant \
	python-dev \
	wget \
	zlib1g-dev

RUN \
 echo "**** fetch tvheadend source ****" && \
 git clone https://github.com/tvheadend/tvheadend.git /tmp/tvheadend

RUN \
 echo "**** compile tvheadend ****" && \
 cd /tmp/tvheadend && \
./configure \
	--build=x86_64-linux-gnu \
	--prefix=/usr \
	--includedir=${prefix}/include \
	--mandir=${prefix}/share/man \
	--infodir=${prefix}/share/info \
	--sysconfdir=/etc \
	--localstatedir=/var \
	--disable-silent-rules \
	--libexecdir=${prefix}/lib/tvheadend \
	--disable-maintainer-mode \
	--disable-dependency-tracking \
	--enable-libffmpeg_static \
	--enable-hdhomerun_static \
	--enable-dvbcsa \
	--enable-bundle \
	--enable-libopus \
	--enable-libvorbis \
	--enable-libvpx \
	--enable-libx264 \
	--enable-libx265 \
	--enable-libav \
	--enable-pngquant \
	--enable-trace \
	--arch=x86_64 && \
 make -j4 && \
 make DESTDIR=/tmp/tvheadend-build install

FROM lsiobase/ubuntu:bionic

# environment settings
ARG DEBIAN_FRONTEND="noninteractive"
ENV HOME="/config"

RUN \
 apt-get update && \
 apt-get install -y \
	bzip2 \
	curl \
	libavahi-client3 \
	libdvbcsa1 \
	liburiparser1 \
	libxmltv-perl \
	pcre2-utils \
	pngquant \
	xmltv-util && \
 echo "**** add picons ****" && \
 mkdir -p \
	/picons && \
 curl -o \
 /tmp/picons.tar.bz2 \
	"https://ci.linuxserver.io/job/Xtras-Builders-Etc/job/picons-builder/lastSuccessfulBuild/artifact/picons.tar.bz2"&& \
 tar xf \
 /tmp/picons.tar.bz2 -C \
	/picons --strip-components=1 && \
 echo "**** cleanup ****" && \
 rm -rf \
	/tmp/* \
	/var/lib/apt/lists/* \
	/var/tmp/*

# copy local files and buildstage artifacts
COPY --from=comskip_buildstage /tmp/binaries/static/bin/comskip /usr/bin/comskip
COPY --from=tvh_buildstage /tmp/tvheadend-build/usr/ /usr/
COPY --from=tvh_buildstage /tmp/tvheadend-build/share/ /usr/share/
COPY root/ /

# ports and volumes
EXPOSE 9981 9982
VOLUME /config /recordings
