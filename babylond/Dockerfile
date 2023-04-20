FROM golang:1.19-alpine3.16 AS build-env

# Customize to your build env

# TARGETPLATFORM should be one of linux/amd64 or linux/arm64.
ARG TARGETPLATFORM

# Use muslc for static libs
ARG BUILD_TAGS=muslc
ARG VERSION
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apk add --update --no-cache make git bash gcc linux-headers eudev-dev ncurses-dev openssh curl jq
RUN apk add --no-cache musl-dev

# Install build dependencies.
RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      wget -P /lib https://github.com/CosmWasm/wasmvm/releases/download/v1.1.2/libwasmvm_muslc.x86_64.a ; \
      cp /lib/libwasmvm_muslc.x86_64.a /lib/libwasmvm_muslc.a ; \
    fi

RUN if  [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      wget -P /lib https://github.com/CosmWasm/wasmvm/releases/download/v1.1.2/libwasmvm_muslc.aarch64.a ; \
      cp /lib/libwasmvm_muslc.aarch64.a /lib/libwasmvm_muslc.a ; \
    fi

# Build
WORKDIR /go/src/github.com/babylonchain
RUN git clone https://github.com/babylonchain/babylon.git
WORKDIR /go/src/github.com/babylonchain/babylon
RUN git fetch
RUN git checkout ${VERSION}
WORKDIR /go/src/github.com/babylonchain/babylon

RUN if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then \
      BUILD_TAGS=${BUILD_TAGS} make install ; \
    fi

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then \
      BUILD_TAGS=${BUILD_TAGS} make install ; \
    fi

# Use busybox to create a user
FROM busybox:stable-musl AS busybox
RUN addgroup --gid 1137 -S defiant && adduser --uid 1137 -S defiant -G defiant

# Use scratch for the final image
FROM scratch
WORKDIR /bin

# Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantchain/babylond:${VERSION}"

# Installs all binaries built with go.
COPY --from=build-env /go/bin/babylond /bin

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
COPY --from=build-env /lib/libcrypto.so.1.1 /lib
COPY --from=build-env /usr/lib/libbrotlidec.so.1 /lib
COPY --from=build-env /usr/lib/libbrotlicommon.so.1 /lib

# Install trusted CA certificates for curl
COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/cert.pem

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


# Copy user from busybox to scratch
COPY --from=busybox /etc/passwd /etc/passwd
COPY --from=busybox --chown=1137:1137 /home/defiant /home/defiant

# Set home directory and user
WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant
