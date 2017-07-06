FROM arm64b/alpine:a9b43ebb524e43a71001006c70921352adcbf6c5 as alpine
RUN \
  apk add \
  bash \
  gcc \
  git \
  go \
  libc-dev \
  libseccomp-dev \
  linux-headers \
  make \
  && true
ENV GOPATH=/root/go
ENV RUNC_COMMIT=ac50e77bbb440dcab354a328c79754e2502b79ca
RUN mkdir -p $GOPATH/src/github.com/opencontainers && \
  cd $GOPATH/src/github.com/opencontainers && \
  git clone https://github.com/opencontainers/runc.git
WORKDIR $GOPATH/src/github.com/opencontainers/runc
RUN git checkout $RUNC_COMMIT
# TODO static pie, currently no easy way to change build options
RUN make static BUILDTAGS="seccomp"
RUN cp runc /usr/bin/

FROM scratch
WORKDIR /
ENTRYPOINT []
COPY --from=alpine /usr/bin/runc /usr/bin/
