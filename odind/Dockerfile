# Declare build arguments at the top of the Dockerfile
ARG ODIN_VERSION=v0.9.4
ARG YODA_VERSION=v0.10.0
ARG ALPINE_VERSION=3.20
ARG UID=1025
ARG USER=heighliner

# Stage 1: Extract the odind binary from the odinprotocol/core image
FROM odinprotocol/core:${ODIN_VERSION} AS odin-source

# Final Stage: Use a specific version of Alpine as the base image
FROM alpine:${ALPINE_VERSION}

# Declare build arguments for reuse
ARG UID
ARG USER
ARG YODA_VERSION

# Set working directory
WORKDIR /home/${USER}

# Install necessary tools and libraries, create user, and set up sudoers
RUN apk add --no-cache \
      curl jq coreutils oniguruma libcurl libgcc nghttp2-libs brotli openssl sudo \
    && addgroup -g ${UID} -S ${USER} \
    && adduser -u ${UID} -S ${USER} -G ${USER} \
    && echo "${USER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${USER} \
    && chmod 0440 /etc/sudoers.d/${USER}

# Use build argument for dynamic labeling
ARG VERSION
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/odind:${VERSION}"

# Copy the odind binary from the source image
COPY --from=odin-source /usr/bin/odind /bin/odind
# COPY --from=odin-source /usr/bin/yoda /bin/yoda

# Download the yoda binary and save it to /bin
RUN curl -L -o /bin/yoda \
      https://github.com/ODIN-PROTOCOL/odin-core/releases/download/${YODA_VERSION}/yoda-${YODA_VERSION}-linux-amd64 \
    && chmod +x /bin/yoda

# Switch to non-root user
USER ${USER}

# Set default entrypoint and command (if applicable)
ENTRYPOINT ["/bin/sh"]
