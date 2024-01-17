FROM golang:1.20-alpine3.16 AS build-env

# TARGETPLATFORM should be one of linux/amd64 or linux/arm64.
ARG TARGETPLATFORM

# Use muslc for static libs
ARG BUILD_TAGS="muslc"
ARG VERSION=v0.9.3-1
ARG ORACLE_VERSION=v0.9.0
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apk update && apk add --no-cache make git bash gcc linux-headers eudev-dev ncurses-dev openssh curl jq
RUN apk add --no-cache musl-dev

# Build Chain Binary
WORKDIR /go/src/github.com/Team-Kujira
RUN git clone https://github.com/Team-Kujira/core.git
WORKDIR /go/src/github.com/Team-Kujira/core
RUN git fetch
RUN git checkout v0.9.3-1
WORKDIR /go/src/github.com/Team-Kujira/core

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
      BUILD_TAGS=${BUILD_TAGS} make install ; \
    fi

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      BUILD_TAGS=${BUILD_TAGS} make install ; \
    fi

# Build Oracle
WORKDIR /go/src/github.com/Team-Kujira
RUN git clone https://github.com/Team-Kujira/oracle-price-feeder.git
WORKDIR /go/src/github.com/Team-Kujira/oracle-price-feeder
RUN git fetch
RUN git checkout ${ORACLE_VERSION}

RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      LEDGER_ENABLED=false GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go install -ldflags ${LD_FLAGS} -tags ${BUILD_TAGS} ; \
    fi

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      LEDGER_ENABLED=false GOOS=linux GOARCH=arm64 CGO_ENABLED=1 go install -ldflags ${LD_FLAGS} -tags ${BUILD_TAGS} ; \
    fi

# Use busybox to create a user
FROM busybox:stable-musl AS busybox
RUN addgroup --gid 1137 -S defiant && adduser --uid 1137 -S defiant -G defiant

# Use scratch for the final image
FROM scratch
WORKDIR /bin

# Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/kujirad:${VERSION}"

# Installs all binaries built with go.
COPY --from=build-env /go/bin/kujirad /bin
COPY --from=build-env /go/bin/price-feeder /bin

# Other binaries we want to keep.
COPY --from=build-env /usr/bin/ldd /bin/ldd
COPY --from=build-env /usr/bin/curl /bin/curl
COPY --from=build-env /usr/bin/jq /bin/jq

# Install Libraries
COPY --from=build-env /usr/lib/libgcc_s.so.1 /lib/
COPY --from=build-env /lib/ld-musl*.so.1* /lib

# jq Libraries
COPY --from=build-env /usr/lib/libonig.so.5 /lib

# curl Libraries
COPY --from=build-env /usr/lib/libcurl.so.4 /lib
COPY --from=build-env /lib/libz.so.1 /lib
COPY --from=build-env /usr/lib/libnghttp2.so.14 /lib

COPY --from=build-env /lib/libssl.so.1.1 /lib
COPY --from=build-env /lib/libcrypto.so.* /lib
COPY --from=build-env /usr/lib/libbrotlidec.so.1 /lib
COPY --from=build-env /usr/lib/libbrotlicommon.so.1 /lib

# Install trusted CA certificates for curl
COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/cert.pem
COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/certs/ca-certificates.crt

# Install cli tools from busybox
COPY --from=busybox /bin/ln /bin/ln
COPY --from=busybox /bin/ln /bin/sh

# Link each binary you want to sh to save space.
RUN for binary in \
dd \
vi \
chown \
id \
cp \
mkdir \
ls \
sh \
cat \
less \
grep \
sleep \
env \
tar \
tee \
du \
sort \
head \
df \
nc \
netstat \
; do ln sh $binary; done


# Copy user
COPY --from=busybox /etc/passwd /etc/passwd
COPY --from=busybox --chown=1137:1137 /home/defiant /home/defiant

# Set home directory and user
WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant
