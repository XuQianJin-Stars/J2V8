ARG sys_image=debian:jessie-slim

FROM $sys_image

RUN mkdir -p /temp/docker/shared/
WORKDIR /temp/docker/shared/

# NOTE: copy shared scripts and run them separately
# this helps when changing commands only in a single script,
# since it will not requrie rebuilding all docker image layers
# but just the ones that were affected

COPY ./shared/install.debian.packages.sh /temp/docker/shared
RUN ./install.debian.packages.sh

ENV ANDROID_HOME "/usr/local/android-sdk"

ENV NDK_VERSION "r14b"
ENV NDK_NAME "android-ndk-$NDK_VERSION-linux-x86_64"
RUN echo "Preparing Android NDK..." && \
    mkdir -p /build && \
    cd /build && \
    curl -fSOL https://dl.google.com/android/repository/$NDK_NAME.zip && \
    unzip $NDK_NAME.zip && \
    rm $NDK_NAME.zip

RUN echo "Preparing Android GCC-Toolchain..." && \
    mkdir -p /build && \
    cd /build && \
    git clone https://github.com/sjitech/android-gcc-toolchain
ENV NDK "/build/android-ndk-$NDK_VERSION"
ENV PATH "$PATH:/build/android-gcc-toolchain:$NDK"

ENV PATH "$PATH:$ANDROID_HOME/tools"
ENV PATH "$PATH:$ANDROID_HOME/platform-tools"

RUN mkdir -p ${ANDROID_HOME} && \
    cd ${ANDROID_HOME} && \
    wget -q https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip -O android_tools.zip && \
    unzip android_tools.zip && \
    rm android_tools.zip

RUN mkdir /usr/local/android-sdk/tools/keymaps && \
    touch /usr/local/android-sdk/tools/keymaps/en-us

COPY ./shared/install.jdk.sh /temp/docker/shared
RUN ./install.jdk.sh
ENV JAVA_HOME "/opt/jdk/jdk1.8.0_131"

ENV PATH ${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools

RUN yes | sdkmanager --licenses

COPY ./shared/install.cmake.sh /temp/docker/shared
RUN ./install.cmake.sh
ENV PATH "$PATH:/opt/cmake/bin"

COPY ./shared/install.gradle.sh /temp/docker/shared
RUN ./install.gradle.sh
ENV GRADLE_HOME "/opt/gradle-3.5"
ENV PATH "$PATH:$GRADLE_HOME/bin"

RUN mkdir -p /temp
COPY ./shared/build.gradle /temp
COPY ./android/AndroidManifest.xml /temp/src/main/AndroidManifest.xml
WORKDIR /temp
RUN cd /temp && gradle --dry-run

# Install the Android tools
RUN sdkmanager emulator
RUN sdkmanager tools
RUN sdkmanager "system-images;android-19;default;armeabi-v7a"

# Required for Android ARM Emulator
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y libqt5widgets5
ENV QT_QPA_PLATFORM offscreen
ENV LD_LIBRARY_PATH ${ANDROID_HOME}/tools/lib64:${ANDROID_HOME}/emulator/lib64:${ANDROID_HOME}/emulator/lib64/qt/lib

EXPOSE 22
EXPOSE 5037
EXPOSE 5554
EXPOSE 5555
EXPOSE 5900

