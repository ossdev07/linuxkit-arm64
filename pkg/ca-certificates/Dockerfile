FROM arm64b/alpine:3f8dd6cd614ccd9c104de391a2cf350adea908c1 as alpine

RUN apk add ca-certificates

FROM scratch
ENTRYPOINT []
WORKDIR /
COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
