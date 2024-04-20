ARG BUILD_FROM
ARG BUILD_VERSION

FROM golang:1.22.2-alpine3.19 AS builder
ARG COREDNS_VERSION=1.11.1
RUN apk add --no-cache \
        git \
        libcap \
        make \
    && git clone --depth 1 --branch v${COREDNS_VERSION} https://github.com/coredns/coredns /coredns
COPY src/plugin.cfg /coredns/
WORKDIR /coredns
RUN go get github.com/coredns/alternate
RUN go get github.com/coredns/records
RUN go generate
# RUN make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARCH=arm64"
ARG BUILD_ARCH
RUN set -x && \
    if [ "${BUILD_ARCH}" = "armhf" ]; then \
        make coredns SYSTEM="CGO_ENABLED=0 GOOS=linux GOARM=6 GOARCH=arm"; \
    elif [ "${BUILD_ARCH}" = "armv7" ]; then \
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
RUN setcap cap_net_bind_service=+ep /coredns/coredns

FROM $BUILD_FROM
EXPOSE 53/tcp \
       53/udp \
       80/tcp \
       443/tcp \
       9153/tcp
COPY --from=builder /coredns/coredns /coredns/
ENTRYPOINT ["/coredns/coredns"]
CMD ["-conf", "/config/Corefile"]
LABEL org.opencontainers.image.authors="robin.mueller@outlook.de" \
      org.opencontainers.image.title="Coredns for Home Assistant" \
      org.opencontainers.image.source="https://github.com/MrRoundRobin/ha_coredns" \
      io.hass.version="2024.4.3" \
      io.hass.type="addon" \
      io.hass.arch="armhf|armv7|aarch64|i386|amd64"
