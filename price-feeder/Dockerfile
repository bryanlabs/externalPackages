FROM golang:1.19 AS builder

# Customize to your build env

# Use muslc for static libs
ARG BUILD_TAGS=netgo
ARG ORACLE_VERSION
ARG KUJIRA_VERSION=v0.7.1
ARG LD_FLAGS=-linkmode=external \
-extldflags '-Wl,-z,muldefs -static'

# Install cli tools for building and final image
RUN apt-get update -y
RUN apt-get install -y make git bash gcc ncurses-dev

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

FROM alpine:latest 
COPY --from=builder /go/bin/price-feeder /usr/local/bin
COPY --from=builder /go/bin/kujirad /usr/local/bin
RUN apk add --no-cache bash
RUN apk add --no-cache gcc
COPY commands.sh /scripts/commands.sh
RUN ["chmod", "+x", "/scripts/commands.sh"]
ENTRYPOINT ["/scripts/commands.sh"]