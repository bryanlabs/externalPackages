FROM golang:1.19 AS build-env

# Customize to your build env

# TARGETPLATFORM should be one of linux/amd64 or linux/arm64.
ARG TARGETPLATFORM

# Use muslc for static libs
ARG BUILD_TAGS=netgo
ARG VERSION
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apt-get update -y
RUN apt-get install -y make git bash gcc ncurses-dev curl jq openssl

# Build
WORKDIR /go/src/github.com/Team-Kujira
RUN git clone https://github.com/Team-Kujira/core.git
WORKDIR /go/src/github.com/Team-Kujira/core
RUN git fetch
RUN git checkout ${VERSION}
WORKDIR /go/src/github.com/Team-Kujira/core

RUN make install 
RUN adduser --system --uid 1137 defiant
RUN addgroup -system --gid 1137 defiant

# # # Label should match your github repo
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/kujirad"

WORKDIR /home/defiant
RUN chown -R defiant /home/defiant
USER defiant
