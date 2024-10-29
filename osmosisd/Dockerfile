# Declare build arguments at the top of the Dockerfile
ARG OSMOSIS_VERSION=26.0.0
ARG ALPINE_VERSION=3.20
ARG UID=1025
ARG USER=heighliner

# Stage 1: Extract the osmosisd binary from the osmolabs/osmosis image
FROM osmolabs/osmosis:${OSMOSIS_VERSION} AS source

# Final Stage: Use a specific version of Alpine as the base image
FROM alpine:${ALPINE_VERSION}
ARG UID
ARG USER
WORKDIR /home/${USER}

# Install necessary tools and libraries in one command, create user
RUN apk add --no-cache curl jq coreutils oniguruma libcurl libgcc nghttp2-libs brotli openssl sudo \
    && addgroup -g ${UID} -S ${USER} \
    && adduser -u ${UID} -S ${USER} -G ${USER} \
    && echo "${USER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${USER} \
    && chmod 0440 /etc/sudoers.d/${USER}

# Use build arguments for dynamic labeling
ARG VERSION
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/osmosisd:${VERSION}"

# Copy the osmosisd binary from the source image
COPY --from=source /bin/osmosisd /bin/osmosisd

# Switch to non-root user
USER ${USER}

# Set default entrypoint and command (optional, if applicable)
ENTRYPOINT ["/bin/sh"]
