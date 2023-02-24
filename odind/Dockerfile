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
RUN apt-get install -y make git bash gcc ncurses-dev curl jq musl-dev

# Install build dependencies.
RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      wget -P /lib https://github.com/CosmWasm/wasmvm/releases/download/v1.1.1/libwasmvm_muslc.x86_64.a ; \
      cp /lib/libwasmvm_muslc.x86_64.a /lib/libwasmvm_muslc.a ; \
    fi

RUN if  [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      wget -P /lib https://github.com/CosmWasm/wasmvm/releases/download/v1.1.1/libwasmvm_muslc.aarch64.a ; \
      cp /lib/libwasmvm_muslc.aarch64.a /lib/libwasmvm_muslc.a ; \
    fi

# Build
WORKDIR /go/src/github.com/ODIN-PROTOCOL
RUN git clone https://github.com/ODIN-PROTOCOL/odin-core.git
WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core
RUN git fetch
RUN git checkout ${VERSION}
# WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core/cmd/odind
WORKDIR /go/src/github.com/ODIN-PROTOCOL/odin-core

RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      # LEDGER_ENABLED=false GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go install -ldflags ${LD_FLAGS} -tags ${BUILD_TAGS} ; \
      # BUILD_TAGS=muslc make install ; \
      make install ; \
    fi

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      # LEDGER_ENABLED=false GOOS=linux GOARCH=arm64 CGO_ENABLED=1 go install -ldflags ${LD_FLAGS} -tags ${BUILD_TAGS} ; \
      BUILD_TAGS=muslc make install ; \
    fi



# Use busybox to create a user
FROM busybox:stable AS busybox
RUN addgroup --gid 1137 -S defiant && adduser --uid 1137 -S defiant -G defiant

# # Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/odin-cored"

# Installs all binaries built with go.
COPY --from=build-env /go/bin /bin

# Install Libraries
COPY --from=build-env /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib/
COPY --from=build-env /lib/x86_64-linux-gnu/libdl.so.2 /lib/
# COPY --from=build-env /go/pkg/mod/github.com/!cosm!wasm/wasmvm@v1.0.0/api/libwasmvm.x86_64.so /lib
# COPY --from=build-env /go/pkg/mod/github.com/slandymani/go-owasm@v0.2.0/api/libgo_owasm.so /lib
COPY --from=build-env /lib/x86_64-linux-gnu/librt.so.1 /lib

# Other binaries we want to keep.
COPY --from=build-env /usr/bin/ldd /bin/ldd
COPY --from=build-env /usr/bin/curl /bin/curl
COPY --from=build-env /usr/bin/jq /bin/jq

# jq Libraries
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libonig.so.5 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libjq.so.1 /lib

# curl Libraries
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libcurl.so.4 /lib
COPY --from=build-env /lib/x86_64-linux-gnu/libz.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libnghttp2.so.14 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libssl.so.1.1 /lib
COPY --from=build-env usr/lib/x86_64-linux-gnu/libcrypto.so.1.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libbrotlidec.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libbrotlicommon.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libidn2.so.0 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/librtmp.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libssh2.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libpsl.so.5 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libgssapi_krb5.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libldap_r-2.4.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/liblber-2.4.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libunistring.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libgnutls.so.30 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libhogweed.so.6 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libnettle.so.8 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libgmp.so.10 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libgcrypt.so.20 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libkrb5.so.3 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libk5crypto.so.3 /lib
COPY --from=build-env /lib/x86_64-linux-gnu/libcom_err.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libkrb5support.so.0 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libsasl2.so.2 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libp11-kit.so.0 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libtasn1.so.6 /lib
COPY --from=build-env /lib/x86_64-linux-gnu/libgpg-error.so.0 /lib
COPY --from=build-env /lib/x86_64-linux-gnu/libkeyutils.so.1 /lib
COPY --from=build-env /usr/lib/x86_64-linux-gnu/libffi.so.7 /lib

WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant
