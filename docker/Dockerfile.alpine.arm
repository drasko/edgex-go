FROM golang:alpine3.9 AS builder
ARG SVC_NAME

WORKDIR /go/src/github.com/edgexfoundry/edgex-go
COPY . .
RUN apk add --update make git \
  && mkdir /tmp/libzmq \
  && if [ "$SVC_NAME" = "core-data" ] || [ "$SVC_NAME" = "export-distro" ]; then \
    apk add libtool pkgconfig build-base autoconf automake util-linux-dev; \
    export PATH=$PATH:`pwd`/docker/toolchain/bin; \
    echo $PATH; \
    arm-linux-musleabihf-gcc --help; \
    wget https://github.com/zeromq/libzmq/archive/v4.3.1.tar.gz; \
    tar -xzf v4.3.1.tar.gz; \
    mv libzmq-4.3.1/* /tmp/libzmq; \
    cd /tmp/libzmq; \
    ./autogen.sh; \
    ./configure --host=arm-linux-musleabihf CC=arm-linux-musleabihf-gcc CXX=arm-linux-musleabihf-g++; \
    make -j 16; \
    cd -; \
    CC=arm-linux-musleabihf-gcc \
    CGO_LDFLAGS=-L/tmp/libzmq/src/.libs \
    CGO_CFLAGS=-I/tmp/libzmq/include \
    GOARCH=arm \
    GOARM=7 \
    PKG_CONFIG_PATH=/tmp/libzmq/src \
    make $SVC_NAME; \
  else \
    GOARCH=arm \
    GOARM=7 \
    make $SVC_NAME; \
  fi \
  && cp -r cmd/$SVC_NAME /edgex \
  && mv /edgex/$SVC_NAME /edgex/exe \
  && echo "$SVC_NAME" > /svc_name 

FROM arm32v7/alpine:3.9.4
COPY --from=builder /go/src/github.com/edgexfoundry/edgex-go/docker/qemu-arm-static /usr/bin/qemu-arm-static
COPY --from=builder /edgex /edgex
COPY --from=builder /svc_name /svc_name
RUN SVC_NAME=`cat /svc_name` \
  && if [ "$SVC_NAME" = "core-data" ] || [ "$SVC_NAME" = "export-distro" ]; then \
    apk update && apk add libzmq; \
  fi \
  && rm -rf /usr/bin/qemu-arm-static

ENTRYPOINT ["/edgex/exe", "--registry", "--profile=docker", "--confdir=/edgex/res"]