FROM rust:slim-bookworm AS source

RUN apt update && apt install -y git pkg-config libssl-dev && \
    git clone https://github.com/sfb103/kellnr.git /tmp/kellnr && \
    cd /tmp/kellnr && git checkout s3-via-serviceacct-w-devcontainer && \
    cargo build --release

##################
# Runtime image  #
##################

FROM rust:slim-bookworm

ARG ADMINPWD=admin
ENV INSTALLDIR=/opt
ENV DATADIR=/opt/kdata
ARG AUTHTOKEN=Zy9HhJ02RJmg0GCrgLfaCVfU6IwDfhXD
ARG VERSION

RUN apt update
RUN apt install -y \ 
  curl \
  sed \
  git \
  zip \
  apt-utils \
  build-essential \
  llvm-dev \
  libclang-dev \
  clang \
  cmake \
  pkg-config \
  libssl-dev

WORKDIR ${INSTALLDIR}
RUN curl -s https://raw.githubusercontent.com/kellnr/installer/main/install.sh | bash -s -- -d ${DATADIR} -p ${ADMINPWD} -t ${AUTHTOKEN} -v ${VERSION} && \
    rm *.zip && \
    rustup component add rust-src

COPY --from=source /tmp/kellnr/target/release/kellnr ${INSTALLDIR}/kellnr/kellnr
COPY --from=source /tmp/kellnr/config/default.toml ${INSTALLDIR}/kellnr/config/default.toml

HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost:8000/api/v1/health || exit 1

EXPOSE 8000

WORKDIR ${INSTALLDIR}/kellnr
ENTRYPOINT ["/bin/sh", "-c" , "update-ca-certificates && ./kellnr"]
