FROM golang:1.19 AS builder

# Customize to your build env

# Use muslc for static libs
ARG BUILD_TAGS=netgo
ARG ORACLE_VERSION=master
ARG KUJIRA_VERSION=v0.7.1
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apt-get update -y
RUN apt-get install -y make git bash gcc ncurses-dev iputils-ping curl jq openssl

# Build Oracle
WORKDIR /go/src/github.com/Team-Kujira
RUN git clone https://github.com/Team-Kujira/oracle-price-feeder.git
WORKDIR /go/src/github.com/Team-Kujira/oracle-price-feeder
RUN git fetch
RUN git checkout ${ORACLE_VERSION}
RUN make install


# Build Kujira
WORKDIR /go/src/github.com/Team-Kujira
RUN git clone https://github.com/Team-Kujira/core.git
WORKDIR /go/src/github.com/Team-Kujira/core
RUN git fetch
RUN git checkout ${KUJIRA_VERSION}
WORKDIR /go/src/github.com/Team-Kujira/core
RUN make install


RUN adduser --system --uid 1137 --shell /bin/bash defiant
RUN addgroup -system --gid 1137 defiant
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/price-feeder"

WORKDIR /home/defiant
COPY ./price-feeder/entrypoint.sh /home/defiant/entrypoint.sh
RUN ["chmod", "+x", "/home/defiant/entrypoint.sh"]
ENTRYPOINT ["/home/defiant/entrypoint.sh"]

RUN chown -R defiant /home/defiant
USER defiant
