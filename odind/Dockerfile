FROM golang:1.19 AS build-env

# Customize to your build env

# TARGETPLATFORM should be one of linux/amd64 or linux/arm64.
ARG TARGETPLATFORM

# Use muslc for static libs
ARG BUILD_TAGS=muslc,netgo
ARG VERSION
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apt-get update -y
RUN apt-get install -y make git bash gcc ncurses-dev curl jq musl-dev openssl

# Build
WORKDIR /go/src/github.com/ODIN-PROTOCOL
RUN git clone https://github.com/ODIN-PROTOCOL/odin-core.git
WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core
RUN git fetch
RUN git checkout ${VERSION}
# WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core/cmd/odind
WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core

# Install CosmWasm.
RUN set -eux; \
    WASM_VERSION=$(go list -m all | grep github.com/CosmWasm/wasmvm | awk '{print $NF}'); \
    echo ${WASM_VERSION} ;\
    if [ ! -z "${WASM_VERSION}" ]; then \
      wget -O /lib/libwasmvm_muslc.a https://github.com/CosmWasm/wasmvm/releases/download/${WASM_VERSION}/libwasmvm_muslc.$(uname -m).a; \
    fi; \
    go mod download;

# Build Chain Binary
RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      make install ; \
    fi

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      make install ; \
    fi

RUN adduser --system --uid 1137 defiant
RUN addgroup -system --gid 1137 defiant

# Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/odind"

WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant
