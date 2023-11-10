# Use the Ubuntu 22.04 base image
FROM ubuntu:22.04

# Set arguments
ARG RELEASE
ARG LAUNCHPAD_BUILD_ARCH

# Set labels
LABEL org.opencontainers.image.ref.name=ubuntu
LABEL org.opencontainers.image.version=22.04

# Set the default shell for the container
SHELL ["/bin/bash", "-c"]

# Install necessary dependencies
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    tzdata curl wget ca-certificates fontconfig locales p11-kit && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    rm -rf /var/lib/apt/lists/*

# Set environment variables
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en LC_ALL=en_US.UTF-8

# Set Java version and download OpenJDK
ENV JAVA_VERSION=jdk-11.0.20.1+1
RUN set -eux; \
    ARCH="$(dpkg --print-architecture)"; \
    case "${ARCH}" in \
        aarch64|arm64) ESUM='0f69f5c05cb7fb2804be3735ed31ce92acff1a51ef29be544b89f83c90d2ea2a'; \
                       BINARY_URL='https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.20.1%2B1/OpenJDK11U-jre_aarch64_linux_hotspot_11.0.20.1_1.tar.gz'; \
                       ;; \
        armhf|arm)     ESUM='2fc1cc935897312c0bc2515b2e7ea1fa3b267e77305a1b51a8c3917d92af380f'; \
                       BINARY_URL='https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.20.1%2B1/OpenJDK11U-jre_arm_linux_hotspot_11.0.20.1_1.tar.gz'; \
                       ;; \
        ppc64el|powerpc:common64) ESUM='7963580e5c3abe55e6b9d2297f2e2cde7b227d28204497bec5f17bb37762c7b7'; \
                       BINARY_URL='https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.20.1%2B1/OpenJDK11U-jre_ppc64le_linux_hotspot_11.0.20.1_1.tar.gz'; \
                       ;; \
        s390x|s390:64-bit) ESUM='cf7fa0f0291687ebcb5f87f5db3a8d94525fd65832adc636c4c6e1f3174d9997'; \
                       BINARY_URL='https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.20.1%2B1/OpenJDK11U-jre_s390x_linux_hotspot_11.0.20.1_1.tar.gz'; \
                       ;; \
        amd64|i386:x86-64) ESUM='bc6ed047e50b09611b419c878e4ea3ea36594bd79f64001a5b53decf72669d33'; \
                       BINARY_URL='https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.20.1%2B1/OpenJDK11U-jre_x64_linux_hotspot_11.0.20.1_1.tar.gz'; \
                       ;; \
        *) echo "Unsupported arch: ${ARCH}"; exit 1; ;; \
    esac; \
    wget -O /tmp/openjdk.tar.gz ${BINARY_URL}; \
    echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c -; \
    mkdir -p "$JAVA_HOME"; \
    tar --extract --file /tmp/openjdk.tar.gz --directory "$JAVA_HOME" --strip-components 1 --no-same-owner ; \
    rm -f /tmp/openjdk.tar.gz ${JAVA_HOME}/lib/src.zip; \
    find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-openjdk.conf; \
    ldconfig; \
    java -Xshare:dump && \
    echo "Verifying install ..." && \
    fileEncoding="$(echo 'System.out.println(System.getProperty(\"file.encoding\"))' | jshell -s -)" && \
    [ "$fileEncoding" = 'UTF-8' ] && \
    rm -rf ~/.java && \
    echo "java --version" && \
    java --version && \
    echo "Complete."

# Copy entrypoint script
COPY file:8b8864b3e02a33a579dc216fd51b28a6047bc8eeaa03045b258980fe0cf7fcb3 /__cacert_entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/__cacert_entrypoint.sh"]

# Set additional environment variables
MAINTAINER zocker-160
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all

# Install additional dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl libxi6 libxrender1 libglu1-mesa libgl1-mesa-glx libxxf86vm1 libxkbcommon0 libsm6 xz-utils netcat screen unzip 7zip curl vim && \
    apt-get autoremove -y && \
    apt-get autoclean -y && \
    apt-get clean -y && \
    apt-get purge -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set working directory
# Set working directory
WORKDIR /sheep

# Verify the existence of files in the build context
RUN ls -l /

# Copy startapp.sh script
COPY startapp.sh /startapp.sh

# Set execute permission for startapp.sh
RUN chmod +x /startapp.sh

# Create a directory for cache
RUN mkdir -p /sheep/cache

# Set environment variables
ENV user_name=zocker_160
ENV user_password=Re85CqEhKAx937dXP19wRf2yRkvAIBemYIVhVaAx
ENV cpu=0
ENV gpu=
ENV ui=text

# Set the default command
CMD ["/startapp.sh"]
