FROM ubuntu:latest
RUN apt-get update
# Install dependencies
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC
RUN yes | apt-get install g++ pkg-config scons ragel gengetopt \
    libuv1-dev libunwind-dev libpulse-dev libsox-dev libcpputest-dev \
    libtool intltool autoconf automake make cmake
RUN yes | apt-get install git wget
RUN yes | apt-get install openssl libssl-dev
RUN yes | apt-get install libspeexdsp-dev
RUN yes | apt-get install libsndfile1 libsndfile1-dev
RUN yes | apt-get install meson
RUN yes | apt-get install ninja-build
RUN yes | apt-get install libtdb-dev check
RUN yes | apt-get install doxygen
# Create Work Directory
RUN mkdir /build
# Build PulseAudio
WORKDIR /build
ENV PULSE_VERSION=17.0
RUN git clone https://gitlab.freedesktop.org/pulseaudio/pulseaudio.git
WORKDIR /build/pulseaudio
RUN git checkout tags/v$PULSE_VERSION
RUN meson setup build
RUN ninja -C build && ninja -C build install
RUN ldconfig
# Build ROC Toolkit
WORKDIR /build
ENV ROCTOOLKIT_VERSION=0.4.0
RUN git clone https://github.com/roc-streaming/roc-toolkit.git
WORKDIR /build/roc-toolkit
RUN git checkout tags/v$ROCTOOLKIT_VERSION
RUN scons -Q --build-3rdparty=openfec && scons -Q --build-3rdparty=openfec install
# Build ROC Pulse
WORKDIR /build
ENV ROCPULSE_VERSION=0.0.7
RUN git clone https://github.com/roc-streaming/roc-pulse.git
WORKDIR /build/roc-pulse
RUN git checkout tags/v$ROCPULSE_VERSION
RUN mkdir build && cd build && mkdir x86_64-linux-gnu
RUN sed -i "1i include(ExternalProject)" "cmake/setup_pulseaudio.cmake"
WORKDIR /build/roc-pulse/build/x86_64-linux-gnu
RUN cmake ../.. \
 -DDOWNLOAD_ROC=OFF \
 -DDOWNLOAD_PULSEAUDIO=OFF \
 -DDOWNLOAD_LIBTOOL=OFF \
 -DPULSEAUDIO_DIR=/build/pulseaudio \
 -DPULSEAUDIO_VERSION=$PULSE_VERSION \
 -DROC_INCLUDE_DIR=/usr/include/roc \
 -DROC_LIB_DIR=/usr/lib/x86_64-linux-gnu
RUN make VERBOSE=1 && make install
# Add ROC input sink to pulseaudio
RUN cp /usr/local/lib/pulse-$PULSE_VERSION/modules/module-roc-sink-input.so /usr/local/lib/x86_64-linux-gnu/pulseaudio/modules
RUN cp /usr/local/lib/pulse-$PULSE_VERSION/modules/module-roc-sink.so /usr/local/lib/x86_64-linux-gnu/pulseaudio/modules
#ENV LD_LIBRARY_PATH=/usr/local/lib/x86_64-linux-gnu/pulseaudio
RUN ldconfig
# Clean-up source
# RUN rm -Rf pulseaudio
# RUN rm -Rf roc-toolkit
# RUN rm -Rf roc-pulse
# Set-up Pulse Audio User
RUN useradd pulse
RUN mkdir /home/pulse
WORKDIR /home/pulse
RUN chgrp pulse .
RUN chmod g+rw .
# Setup PulseAudio start up script
RUN echo "#!/bin/sh" > launch.sh
RUN echo "echo \"load-module module-tunnel-sink sink_name=Remote server=\$PULSE_SERVER\" >> /home/pulse/config.pa" >> launch.sh
RUN echo "echo \"load-module module-roc-sink-input local_ip=0.0.0.0\" >> /home/pulse/config.pa" >> launch.sh
RUN echo "rm -Rf /tmp/pulse-*" >> launch.sh
RUN echo "/usr/local/bin/pulseaudio -v --exit-idle-time=-1 -F /home/pulse/config.pa --exit-idle-time=-1" >> launch.sh
RUN chmod g+x launch.sh
# Start PulseAudio
USER pulse
CMD ["/home/pulse/launch.sh"]
#CMD ["/bin/bash"]
#CMD ["tail", "-f", "/dev/null"]