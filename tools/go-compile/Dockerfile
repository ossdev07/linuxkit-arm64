FROM arm64b/alpine:a9b43ebb524e43a71001006c70921352adcbf6c5 AS mirror
RUN mkdir -p /out/etc/apk && cp -r /etc/apk/* /out/etc/apk/
RUN apk add --no-cache --initdb -p /out \
    alpine-baselayout \
    busybox \
    git \
    go \
    musl-dev
RUN rm -rf /out/etc/apk /out/lib/apk /out/var/cache

FROM scratch
ENV GOPATH=/go PATH=$PATH:/go/bin
COPY --from=mirror /out/ /
COPY --from=mirror /go/bin/ /go/bin/
COPY /compile.sh /compile.sh
ENTRYPOINT ["/compile.sh"]
