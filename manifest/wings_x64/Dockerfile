# Base build stage
FROM golang:1.21-alpine as build

ARG VERSION
ARG WITH_KVM=false

RUN apk add --no-cache git

# Clone the repository
WORKDIR /wings
RUN git clone https://github.com/pterodactyl/wings --depth 1 --branch ${VERSION} .

# Copy and apply the patch if needed using git apply
COPY kvm_patch.patch /tmp/kvm_patch.patch
RUN if [ "$WITH_KVM" = "true" ]; then \
        git apply /tmp/kvm_patch.patch; \
    fi
RUN rm /tmp/kvm_patch.patch

# Build the application
RUN BuildVersion=$(echo "$VERSION" | sed 's/v//') \
    && CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
        -ldflags="-s -w -X github.com/pterodactyl/wings/system.Version=$BuildVersion" \
        -v \
        -trimpath \
        -o wings \
        wings.go

# Final stage
FROM alpine:latest as final
LABEL maintainer="Phoenix"

RUN apk add --no-cache ca-certificates tini tzdata \
    && mkdir -p /etc/pterodactyl /var/log/pterodactyl /tmp/pterodactyl

# Copy the application binary
COPY --from=build /wings/wings /usr/local/bin

# Copy additional files
COPY ./root /
RUN chmod 755 /entrypoint.sh

EXPOSE 8080
EXPOSE 2022

RUN addgroup -g 3000 pterodactyl \
    && adduser \
    --disabled-password \
    --gecos "" \
    --ingroup "pterodactyl" \
    --no-create-home \
    --uid "3000" \
    "pterodactyl"

# Default Work Directory
WORKDIR /var/lib/pterodactyl

ENTRYPOINT [ "tini", "--", "/entrypoint.sh" ]
