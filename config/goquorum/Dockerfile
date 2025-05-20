# -------------------------
# Dockerfile from quorum, commit 8585ac91f
# -------------------------
# Build Geth in a stock Go builder container
FROM golang:1.22-alpine as builder

RUN apk add --no-cache gcc musl-dev linux-headers git

# Get dependencies - will also be cached if we won't change go.mod/go.sum
COPY quorum-code/go.mod /go-ethereum/
COPY quorum-code/go.sum /go-ethereum/
RUN cd /go-ethereum && go mod download

ADD quorum-code/ /go-ethereum/
RUN cd /go-ethereum && go run build/ci.go install -static ./cmd/geth
RUN cd /go-ethereum && go run build/ci.go install -static ./cmd/bootnode

# Pull Geth into a second stage deploy alpine container
FROM alpine:latest

RUN apk add --no-cache ca-certificates curl
RUN apk add --no-cache openssl # quorum (6 may 2024): 3.1.4-r5 is the installed openssl version, want 3.1.4-r6 to get fix for CVE-2024-2511
COPY --from=builder /go-ethereum/build/bin/geth /usr/local/bin/
COPY --from=builder /go-ethereum/build/bin/bootnode /usr/local/bin/

EXPOSE 8545 8546 30303 30303/udp

# -------------------------
# Dockerfile from dev quickstart
# -------------------------
COPY docker-entrypoint.sh /usr/local/bin/
COPY data data

RUN mkdir -p /data /permissions /var/log/quorum \
    && addgroup -g 1000 quorum \
    && adduser -u 1000 -G quorum -s /bin/sh -D quorum \
    && chown -R quorum:quorum /data /permissions /var/log/quorum 
USER quorum

ENTRYPOINT ["docker-entrypoint.sh"]
