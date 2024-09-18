FROM golang:1.22.7-alpine3.20 AS build-env

# Use muslc for static libs
ARG BUILD_TAGS=muslc
ARG VERSION
# ARG LD_FLAGS=-linkmode=external \
# -extldflags '-Wl,-z,muldefs -static'
RUN export GOOS=linux GOARCH=$TARGETARCH CGO_ENABLED=1 LDFLAGS='-linkmode external -extldflags "-static"'
# Install cli tools for building and final image
RUN apk add --update --no-cache make git bash gcc linux-headers eudev-dev ncurses-dev openssh curl jq coreutils
RUN apk add --no-cache musl-dev

# Build
WORKDIR /go/src/github.com/osmosis-labs
RUN git clone https://github.com/osmosis-labs/osmosis.git
WORKDIR /go/src/github.com/osmosis-labs/osmosis
RUN git fetch
RUN git checkout ${VERSION}
WORKDIR /go/src/github.com/osmosis-labs/osmosis

# Install CosmWasm.
RUN set -eux; \
    WASM_VERSION=$(go list -m all | grep github.com/CosmWasm/wasmvm | awk '{print $NF}'); \
    echo ${WASM_VERSION}; \
    wget -O /lib/libwasmvm_muslc.x86_64.a https://github.com/CosmWasm/wasmvm/releases/download/${WASM_VERSION}/libwasmvm_muslc.x86_64.a; \
    go mod download;

RUN echo ${BUILD_TAGS}

# Build Chain Binary for amd64
RUN BUILD_TAGS=${BUILD_TAGS} make install

# # Use busybox to create a user
# FROM busybox:stable-musl AS busybox
# RUN addgroup --gid 1137 -S defiant && adduser --uid 1137 -S defiant -G defiant

# # Use scratch for the final image
# FROM scratch
# WORKDIR /bin
# ARG VERSION

# # Label should match your GitHub repo
# LABEL org.opencontainers.image.source="https://github.com/defiantlabs/osmosisd:${VERSION}"

# # Copy the built binary
# COPY --from=build-env /go/bin/osmosisd /bin

# # Copy other necessary binaries
# COPY --from=build-env /usr/bin/ldd /bin/ldd
# COPY --from=build-env /usr/bin/curl /bin/curl
# COPY --from=build-env /usr/bin/jq /bin/jq
# COPY --from=build-env /usr/bin/tac /bin/tac

# # Install Libraries
# COPY --from=build-env /usr/lib/libgcc_s.so.1 /lib/
# COPY --from=build-env /lib/ld-musl*.so.1* /lib
# COPY --from=build-env /usr/lib/libonig.so.5 /lib
# COPY --from=build-env /lib/libacl.so* /lib
# COPY --from=build-env /lib/libattr.so* /lib
# COPY --from=build-env /lib/libutmps.so* /lib
# COPY --from=build-env /lib/libskarnet.so* /lib
# COPY --from=build-env /usr/lib/libcurl.so.4 /lib
# COPY --from=build-env /lib/libz.so.1 /lib
# COPY --from=build-env /usr/lib/libnghttp2.so.14 /lib
# COPY --from=build-env /lib/libssl.so* /lib
# COPY --from=build-env /lib/libcrypto.so* /lib
# COPY --from=build-env /usr/lib/libbrotlidec.so.1 /lib
# COPY --from=build-env /usr/lib/libbrotlicommon.so.1 /lib

# # Install trusted CA certificates for curl
# COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/cert.pem
# COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/ca-certificates.crt
# COPY --from=build-env /etc/ssl/cert.pem /etc/ssl/certs/ca-certificates.crt

# # Install cli tools from busybox
# COPY --from=busybox /bin/ln /bin/ln
# COPY --from=busybox /bin/ln /bin/sh

# # Link each binary you want to sh to save space.
# RUN for binary in \
# dd \
# vi \
# chown \
# id \
# cp \
# ls \
# sh \
# cat \
# less \
# grep \
# sleep \
# env \
# tar \
# tee \
# du \
# sort \
# head \
# df \
# nc \
# netstat \
# ; do ln sh $binary; done

# # Copy user
# COPY --from=busybox /etc/passwd /etc/passwd
# COPY --from=busybox --chown=1137:1137 /home/defiant /home/defiant

# # Set home directory and user
# WORKDIR /home/defiant
# RUN chown -R defiant /home/defiant
# USER defiant
