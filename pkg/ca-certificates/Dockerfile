FROM arm64b/alpine:a9b43ebb524e43a71001006c70921352adcbf6c5 as alpine

RUN apk add ca-certificates

FROM scratch
ENTRYPOINT []
WORKDIR /
COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
