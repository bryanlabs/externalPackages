FROM golang:1.19-buster AS build-env

ARG TARGETPLATFORM
ARG BUILD_TAGS=netgo
ARG VERSION
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apt-get update && \
    apt-get install -y make git bash gcc linux-headers-amd64 libudev-dev libncurses-dev openssh-client curl jq

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

# Use Debian Buster slim for the final image
FROM debian:buster-slim

WORKDIR /bin

LABEL org.opencontainers.image.source="https://github.com/defiantchain/osmosisd:${VERSION}"

COPY --from=build-env /go/bin/odind /bin
COPY --from=build-env /go/bin/yoda /bin
COPY --from=build-env /usr/bin/ldd /bin/ldd
COPY --from=build-env /usr/bin/curl /bin/curl
COPY --from=build-env /usr/bin/jq /bin/jq
COPY --from=build-env /go/pkg/mod/github.com/slandymani/go-owasm@v0.2.0/api/libgo_owasm.so /usr/lib/
COPY --from=build-env /go/pkg/mod/github.com/!cosm!wasm/wasmvm@v1.0.0/api/libwasmvm.x86_64.so /usr/lib/

# RUN mkdir -p /etc/ssl/certs/
RUN apt-get update && apt-get install -y ca-certificates
# COPY --from=build-env /etc/ssl/certs/ca-certificates.crt /etc/ssl/cert.pem
# COPY --from=build-env /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt


RUN adduser --system --uid 1137 defiant
RUN addgroup -system --gid 1137 defiant

# Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/odind"

WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant