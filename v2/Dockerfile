#################
# Build Step
#################

FROM golang:alpine as build
LABEL maintainers="Ben Yanke <ben@benyanke.com>, Jörn Friedrich Dreyer <jfd@butonic.de>, Chris F Ravenscroft <chris@voilaweb.com>"

# Setup work env
RUN mkdir /app /tmp/gocode
ADD local/app /app/
WORKDIR /app/v2

# Required envs for GO
ENV GOPATH=/tmp/gocode
ENV GOOS=linux
ENV GOARCH=amd64
ENV CGO_ENABLED=0
ENV GOPROXY=https://goproxy.cn,direct

# Only needed for alpine builds // also: busybox
RUN apk add --no-cache git make busybox-static dumb-init gcc g++
ADD go.mod go.sum  ./
ADD ./vendored/ ./vendored

# Install deps
RUN go mod download

ADD . .

# Build and copy final result
RUN uname -a
RUN if [ $(uname -m) == x86_64 ]; then CGO_ENABLED=1 make linuxamd64 && CGO_ENABLED=1 make plugins_linux_amd64 && cp ./bin/linuxamd64/glauth /app/v2/glauth && cp ./bin/linuxamd64/*.so /app/v2/; fi
RUN if [ $(uname -m) == aarch64 ]; then CGO_ENABLED=1 make linuxarm64 && cp ./bin/linuxarm64/glauth /app/v2/glauth; fi
RUN if [ $(uname -m) == armv7l ]; then make linuxarm && cp ./bin/linuxarm/glauth /app/v2/glauth; fi


# Check glauth works
RUN /app/v2/glauth --version

#################
# Run Step
#################

FROM alpine:latest as run

# Copy binary from build container
COPY --from=build /app/v2/glauth /app/glauth
COPY --from=build /app/v2/*.so /app/


# Copy docker specific scripts from build container
COPY --from=build /app/v2/scripts/docker/start-standalone.sh /app/docker/
COPY --from=build /app/v2/scripts/docker/default-config-standalone.cfg /app/docker/

# Just what we need
COPY --from=build /usr/bin/dumb-init /usr/bin/dumb-init


# Expose web and LDAP ports
EXPOSE 389 636 5555

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/bin/sh", "/app/docker/start-standalone.sh"]
