# Build Lighthouse in a stock Rust build container
FROM rust:latest as builder

ARG BUILD_TARGET

RUN apt-get update && apt-get install -y git gcc g++ make cmake pkg-config libssl-dev

WORKDIR /usr/src
RUN git clone https://github.com/sigp/lighthouse.git && cd lighthouse && git config advice.detachedHead false && git fetch --all --tags && git checkout ${BUILD_TARGET} && make

# Pull all binaries into a second stage deploy debian container
FROM debian:buster-slim

ARG USER
ARG UID

RUN apt-get update && apt-get install -y --no-install-recommends \
  libssl-dev \
  ca-certificates \
  wget \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Grab dockerize so beacon can wait for geth
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# See https://stackoverflow.com/a/55757473/12429735RUN
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    "${USER}"

# Create data mount point with permissions
RUN mkdir -p /var/lib/lighthouse && chown ${USER}:${USER} /var/lib/lighthouse
# Copy executable
COPY --from=builder /usr/local/cargo/bin/lighthouse /usr/local/bin/
# Script to query whether all validator key passwords are the same
COPY ./query-lh-validator-passwords.sh /usr/local/bin
# Use an unprivileged user.
USER ${USER}:${USER}

#EXPOSE 5052 9000 9000/udp
ENTRYPOINT ["lighthouse", "bn", "--datadir", "/var/lib/lighthouse/", "--http", "--http-address", "0.0.0.0"]
CMD ["--testnet", "medalla", "--eth1-endpoint", "http://geth:8545"]
