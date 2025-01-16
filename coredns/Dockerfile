ARG BUILD_FROM
ARG BUILD_VERSION

FROM golang:1.22 AS builder
ARG COREDNS_VERSION=1.12.0
RUN apt-get update;\
    apt-get install -y --no-install-recommends git libcap2-bin; \
    rm -rf /var/lib/apt/lists/*
RUN git clone --depth 1 --branch v${COREDNS_VERSION} https://github.com/coredns/coredns /go/src/github.com/coredns/coredns
COPY src/plugin.cfg /go/src/github.com/coredns/coredns
WORKDIR /go/src/github.com/coredns/coredns
RUN go get github.com/coredns/alternate
RUN go get github.com/coredns/records
RUN go generate
ARG BUILD_ARCH
RUN set -x && \
    if [ "${BUILD_ARCH}" = "armv7" ]; then \
        make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARM=7 GOARCH=arm"; \
    elif [ "${BUILD_ARCH}" = "aarch64" ]; then \
        make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARCH=arm64"; \
    elif [ "${BUILD_ARCH}" = "i386" ]; then \
        make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARCH=386"; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then \
        make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARCH=amd64"; \
    else \
        exit 1; \
    fi
RUN setcap cap_net_bind_service=+ep /go/src/github.com/coredns/coredns/coredns

FROM $BUILD_FROM
EXPOSE 53/tcp \
       53/udp \
       80/tcp \
       443/tcp \
       9153/tcp
COPY --from=builder /go/src/github.com/coredns/coredns/coredns /coredns/
ENTRYPOINT ["/coredns/coredns"]
CMD ["-conf", "/config/Corefile"]
LABEL org.opencontainers.image.authors="robin.mueller@outlook.de" \
      org.opencontainers.image.title="CoreDNS for Home Assistant" \
      org.opencontainers.image.source="https://github.com/MrRoundRobin/ha_coredns" \
      io.hass.version="2025.1.0" \
      io.hass.type="addon" \
      io.hass.arch="armhf|armv7|aarch64|i386|amd64"
