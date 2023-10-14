FROM golang:1.20.10-alpine as build

ARG VERSION

RUN \
    apk add git \
    && cd / \ 
    && git clone https://github.com/pterodactyl/wings --depth 1 --branch ${VERSION} \
    && cd wings \
    && BuildVersion=$(echo "$VERSION" | sed 's/v//') \
    && CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build \
        -ldflags="-s -w -X github.com/pterodactyl/wings/system.Version=$BuildVersion" \
        -v \
        -trimpath \
        -o wings \
        wings.go

FROM alpine:latest as final
LABEL maintainer="Phoenix"

RUN \
    apk add --no-cache ca-certificates tini tzdata \
    && mkdir -p /etc/pterodactyl /var/log/pterodactyl /tmp/pterodactyl 

# Copy wings from build stage
COPY --from=build /wings/wings /usr/local/bin
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
