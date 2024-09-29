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
# Build PulseAudio
RUN wget https://freedesktop.org/software/pulseaudio/releases/pulseaudio-15.99.1.tar.gz
RUN tar -xzvf pulseaudio-15.99.1.tar.gz
RUN rm pulseaudio-15.99.1.tar.gz
RUN cd pulseaudio-15.99.1 && meson setup build
RUN cd pulseaudio-15.99.1 && ninja -C build
RUN cd pulseaudio-15.99.1 && ninja -C build install
RUN cd pulseaudio-15.99.1 && ldconfig
# Build Libtool
RUN wget https://gnu.askapache.com/libtool/libtool-2.4.7.tar.gz
RUN tar -xzvf libtool-2.4.7.tar.gz
RUN rm libtool-2.4.7.tar.gz
RUN cd libtool-2.4.7 && ./configure --disable-static
RUN cd libtool-2.4.7 && make
RUN cd libtool-2.4.7 && make install
# Build ROC Toolkit
RUN git clone https://github.com/roc-streaming/roc-toolkit.git
RUN cd roc-toolkit && scons -Q --build-3rdparty=openfec
RUN cd roc-toolkit && scons -Q --build-3rdparty=openfec install
# Build ROC Pulse
RUN git clone https://github.com/roc-streaming/roc-pulse.git
RUN cd roc-pulse && mkdir build && cd build && mkdir x86_64-linux-gnu
RUN sed -i "1i include(ExternalProject)" "/roc-pulse/cmake/setup_pulseaudio.cmake"
RUN cd roc-pulse/build/x86_64-linux-gnu && cmake ../.. \
 -DDOWNLOAD_ROC=OFF \
 -DDOWNLOAD_PULSEAUDIO=OFF \
 -DDOWNLOAD_LIBTOOL=OFF \
 -DPULSEAUDIO_DIR=/pulseaudio-15.99.1 \
 -DPULSEAUDIO_VERSION=15.99.1 \
 -DROC_INCLUDE_DIR=/usr/include/roc \
 -DROC_LIB_DIR=/roc-toolkit/bin/x86_64-pc-linux-gnu \
 -DLIBTOOL_VERSION=2.4.7 \
 -DLIBTOOL_DIR=/libtool-2.4.7
RUN cd roc-pulse/build/x86_64-linux-gnu && make VERBOSE=1
RUN cd roc-pulse/build/x86_64-linux-gnu && make install
# Add ROC input sink to pulseaudio
RUN cp /usr/local/lib/pulse-15.99.1/modules/module-roc-sink-input.so /usr/local/lib/x86_64-linux-gnu/pulseaudio/modules
RUN cp /usr/local/lib/pulse-15.99.1/modules/module-roc-sink.so /usr/local/lib/x86_64-linux-gnu/pulseaudio/modules
ENV LD_LIBRARY_PATH=/usr/local/lib/x86_64-linux-gnu/pulseaudio
RUN ldconfig
# Clean-up source
# RUN rm -Rf pulseaudio-15.99.1
# RUN rm -Rf libtool-2.4.7
# RUN rm -Rf roc-toolkit
# RUN rm -Rf roc-pulse
# Setup PulseAudio start up script
RUN mkdir /run/pulse
RUN echo "#!/bin/sh" > /run/pulse/launch.sh
RUN echo "echo \"load-module module-tunnel-sink sink_name=Remote server=\$PULSE_SERVER\" >> /run/pulse/config.pa" >> /run/pulse/launch.sh
RUN echo "echo \"load-module module-roc-sink-input local_ip=0.0.0.0\" >> /run/pulse/config.pa" >> /run/pulse/launch.sh
RUN echo "rm -Rf /tmp/pulse-*" >> /run/pulse/launch.sh
RUN echo "/usr/local/bin/pulseaudio -vvv --exit-idle-time=-1 -F /run/pulse/config.pa --exit-idle-time=-1" >> /run/pulse/launch.sh
RUN chmod +x "/run/pulse/launch.sh"
# Start PulseAudio
CMD ["/run/pulse/launch.sh"]
#CMD ["/bin/bash"]
#CMD ["tail", "-f", "/dev/null"]