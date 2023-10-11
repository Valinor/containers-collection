FROM ubuntu:22.04

ENV ANDROID_HOME="/opt/android-sdk"

# support amd64 and arm64
RUN JDK_PLATFORM=$(if [ "$(uname -m)" = "aarch64" ]; then echo "arm64"; else echo "amd64"; fi) && \
    echo export JDK_PLATFORM=$JDK_PLATFORM >> /etc/jdk.env && \
    echo export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-$JDK_PLATFORM/" >> /etc/jdk.env && \
    echo . /etc/jdk.env >> /etc/bash.bashrc && \
    echo . /etc/jdk.env >> /etc/profile

ENV TZ=America/Los_Angeles

# Get the latest version from https://developer.android.com/studio/index.html
ENV ANDROID_SDK_TOOLS_VERSION="4333796"

# Set locale
ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8" \
    LC_ALL="en_US.UTF-8"

RUN apt-get clean && \
    apt-get update -qq && \
    apt-get install -qq -y apt-utils locales && \
    locale-gen $LANG

ENV DEBIAN_FRONTEND="noninteractive" \
    TERM=dumb \
    DEBIAN_FRONTEND=noninteractive

# Variables must be references after they are created
ENV ANDROID_SDK_HOME="$ANDROID_HOME"

ENV PATH="$JAVA_HOME/bin:$PATH:$ANDROID_SDK_HOME/emulator:$ANDROID_SDK_HOME/cmdline-tools/bin:$ANDROID_SDK_HOME/cmdline-tools:$ANDROID_SDK_HOME/platform-tools:/opt/gradle/gradle-8.4/bin"

WORKDIR /tmp

# Installing packages
RUN apt-get update -qq > /dev/null && \
    apt-get install -qq locales > /dev/null && \
    locale-gen "$LANG" > /dev/null && \
    apt-get install -qq --no-install-recommends \
        openjdk-17-jdk \
        wget \
        unzip \
        zip \
        zipalign > /dev/null && \
    echo "JVM directories: `ls -l /usr/lib/jvm/`" && \
    . /etc/jdk.env && \
    echo "Java version (default):" && \
    java -version && \
    echo "set timezone" && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    rm -rf /tmp/* /var/tmp/*

# Install Android SDK
RUN echo "sdk tools ${ANDROID_SDK_TOOLS_VERSION}" && \
    wget --quiet --output-document=sdk-tools.zip \
        "https://dl.google.com/android/repository/commandlinetools-linux-10406996_latest.zip" && \
#        "https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip" && \
    mkdir --parents "$ANDROID_HOME" && \
    unzip -q sdk-tools.zip -d "$ANDROID_HOME" && \
    rm --force sdk-tools.zip

# Install SDKs
# Please keep these in descending order!
# The `yes` is for accepting all non-standard tool licenses.
RUN mkdir --parents "$ANDROID_HOME/.android/" && \
    echo '### User Sources for Android SDK Manager' > \
        "$ANDROID_HOME/.android/repositories.cfg" && \
    . /etc/jdk.env && \
    yes | "$ANDROID_HOME"/cmdline-tools/bin/sdkmanager --licenses --sdk_root=/opt/android-sdk/ > /dev/null

# List all available packages.
# redirect to a temp file `packages.txt` for later use and avoid show progress
RUN . /etc/jdk.env && \
    "$ANDROID_HOME"/cmdline-tools/bin/sdkmanager --list --sdk_root=/opt/android-sdk/ > packages.txt && \
    cat packages.txt | grep -v '='

#
# https://developer.android.com/studio/command-line/sdkmanager.html
#
RUN echo "platforms" && \
    . /etc/jdk.env && \
    yes | "$ANDROID_HOME"/cmdline-tools/bin/sdkmanager --sdk_root=/opt/android-sdk/ \
        "platforms;android-32" > /dev/null

RUN echo "platform tools" && \
    . /etc/jdk.env && \
    yes | "$ANDROID_HOME"/cmdline-tools/bin/sdkmanager --sdk_root=/opt/android-sdk/\
        "platform-tools" > /dev/null

RUN echo "build tools 26-30" && \
    . /etc/jdk.env && \
    yes | "$ANDROID_HOME"/cmdline-tools/bin/sdkmanager --sdk_root=/opt/android-sdk/\
        "build-tools;31.0.0" > /dev/null

RUN echo "bundletool" && \
    wget -q https://github.com/google/bundletool/releases/download/1.15.4/bundletool-all-1.15.4.jar -O bundletool.jar && \
    mv bundletool.jar $ANDROID_SDK_HOME/cmdline-tools/

RUN du -sh $ANDROID_HOME

# Copy sdk license agreement files.
#RUN mkdir -p $ANDROID_HOME/licenses
#COPY sdk/licenses/* $ANDROID_HOME/licenses/
RUN wget https://services.gradle.org/distributions/gradle-8.4-bin.zip &&\
    mkdir /opt/gradle &&\
    unzip -d /opt/gradle gradle-8.4-bin.zip &&\
    rm gradle-8.4-bin.zip


WORKDIR /project
