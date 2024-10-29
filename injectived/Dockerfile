ARG CHAIN_VERSION=v1.13.2
ARG DEBIAN_VERSION=bookworm-slim
ARG UID=1025
ARG USER=heighliner

# Stage 1: Extract the source binary.
FROM public.ecr.aws/l9h3g6c6/injective-core:${CHAIN_VERSION} AS source

# Final Stage: Use a specific version of Debian as the base image
FROM debian:${DEBIAN_VERSION}
ARG UID
ARG USER
WORKDIR /home/${USER}

# Install necessary tools and libraries, create user
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl jq coreutils libonig5 libcurl4 libgcc1 libnghttp2-14 brotli openssl sudo \
    && groupadd -g ${UID} ${USER} \
    && useradd -u ${UID} -m -g ${USER} -s /bin/bash ${USER} \
    && echo "${USER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${USER} \
    && chmod 0440 /etc/sudoers.d/${USER} \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Use build arguments for dynamic labeling
ARG VERSION
LABEL org.opencontainers.image.source="https://github.com/defiantlabs/injectived:${VERSION}"

# Copy the injectived binary and required library from the source image
COPY --from=source /usr/local/bin/injectived /bin/injectived
COPY --from=source /usr/lib/libwasmvm.x86_64.so /usr/lib/libwasmvm.x86_64.so
# Set correct permissions for the library
RUN chmod 755 /usr/lib/libwasmvm.x86_64.so

# Set the library path for runtime linking
ENV LD_LIBRARY_PATH=/usr/lib

# Switch to non-root user
USER ${USER}

# Set default entrypoint and command (optional, if applicable)
ENTRYPOINT ["/bin/bash"]
