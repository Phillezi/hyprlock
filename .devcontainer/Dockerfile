FROM debian:bookworm AS builder

ENV DEBIAN_FRONTEND=noninteractive

# Install build tools and dependencies
RUN echo 'deb http://deb.debian.org/debian testing main' >> /etc/apt/sources.list && \
    apt update && apt install -y \
        git \
        curl \
        build-essential \
        gcc-13 g++-13 \
        cmake \
        meson \
        ninja-build \
        pkg-config \
        python3-pip \
        python3-jinja2 \
        python3-markupsafe \
        libexpat1-dev \
        libwayland-dev \
        wayland-protocols \
        libxkbcommon-dev \
        libpango1.0-dev \
        libcairo2-dev \
        libpam0g-dev \
        libgbm-dev \
        libdrm-dev \
        libgl1-mesa-dev \
        libegl1-mesa-dev \
        libsystemd-dev \
        libjpeg-dev \
        libwebp-dev \
        libmagic-dev \
        libpugixml-dev && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# --- Build sdbus-cpp ---
RUN git clone --recursive https://github.com/Kistler-Group/sdbus-cpp.git && \
    cd sdbus-cpp && \
    git checkout v2.1.0 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr && \
    make -j$(nproc) && make install

# --- Build hyprutils ---
RUN git clone https://github.com/hyprwm/hyprutils.git && \
    cd hyprutils && cmake \
                        --no-warn-unused-cli \
                        -DCMAKE_BUILD_TYPE:STRING=Release \
                        -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . \
                        -B ./build && \
                        cmake \
                        --build ./build \
                        --config Release \
                        --target all \
                        -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF` && \
                        cmake --install build

# --- Build hyprlang ---
RUN git clone https://github.com/hyprwm/hyprlang.git && \
    cd hyprlang && cmake \
                        --no-warn-unused-cli \
                        -DCMAKE_BUILD_TYPE:STRING=Release \
                        -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . \
                        -B ./build && \
                        cmake \
                        --build ./build \
                        --config Release \
                        --target hyprlang \
                        -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF` && \
                        cmake --install ./build

# --- Build hyprgraphics ---
RUN git clone https://github.com/hyprwm/hyprgraphics.git && \
    cd hyprgraphics && cmake \
                        --no-warn-unused-cli \
                        -DCMAKE_BUILD_TYPE:STRING=Release \
                        -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . \
                        -B ./build && \
                        cmake \
                        --build ./build \
                        --config Release \
                        --target all \
                        -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF` && \
                        cmake --install build

# --- Build hyprwayland-scanner ---
RUN git clone https://github.com/hyprwm/hyprwayland-scanner.git && \
    cd hyprwayland-scanner && cmake \
                                -DCMAKE_INSTALL_PREFIX=/usr \
                                -B build && \
                                cmake \
                                --build build \
                                -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF` && \
                                cmake --install build

# --- Install hyprland-protocols ---
RUN git clone https://github.com/hyprwm/hyprland-protocols.git && \
    cd hyprland-protocols && meson setup build && \
                            meson install -C build

FROM mcr.microsoft.com/devcontainers/cpp:1-debian-12

RUN echo 'deb http://deb.debian.org/debian testing main' >> /etc/apt/sources.list && \
    apt update && export DEBIAN_FRONTEND=noninteractive \
    && apt -y install --no-install-recommends \
                                    build-essential \
                                    gcc-13 g++-13 \
                                    pkg-config \
                                    libwayland-dev \
                                    wayland-protocols \
                                    libxkbcommon-dev \
                                    libpango1.0-dev \
                                    libcairo2-dev \
                                    libpam0g-dev \
                                    libgbm-dev \
                                    libdrm-dev \
                                    libgl1-mesa-dev \
                                    libegl1-mesa-dev \
                                    libsystemd-dev \
                                    libaudit-dev \
                                    git \
                                    curl && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

ARG REINSTALL_CMAKE_VERSION_FROM_SOURCE="3.27.0"

COPY ./reinstall-cmake.sh /tmp/

RUN if [ "${REINSTALL_CMAKE_VERSION_FROM_SOURCE}" != "none" ]; then \
        chmod +x /tmp/reinstall-cmake.sh && /tmp/reinstall-cmake.sh ${REINSTALL_CMAKE_VERSION_FROM_SOURCE}; \
    fi \
    && rm -f /tmp/reinstall-cmake.sh

COPY --from=builder /usr/local /usr/local
COPY --from=builder /usr/lib /usr/lib
COPY --from=builder /usr/include /usr/include
COPY --from=builder /usr/bin /usr/bin
