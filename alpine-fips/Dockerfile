FROM alpine:3.9 AS build

LABEL maintainer="Mikhail Merkulov <mihail.merkulov@gmail.com>"

ARG OPENSSL_FIPS_VER=2.0.16
ARG OPENSSL_FIPS_HMACSHA1=e8dbfa6cb9e22a049ec625ffb7ccaf33e6116598
ARG OPENSSL_FIPS_HASH=a3cd13d0521d22dd939063d3b4a0d4ce24494374b91408a05bdaca8b681c63d4
ARG OPENSSL_FIPS_PGP_FINGERPRINT=D3577507FA40E9E2
ARG OPENSSL_VER=1.0.2o
ARG OPENSSL_HASH=ec3f5c9714ba0fd45cb4e087301eb1336c317e0d20b575a125050470e8089e4d
ARG OPENSSL_PGP_FINGERPRINT=D9C4D26D0E604491

WORKDIR /tmp/build

#ADD openssl-fips-${OPENSSL_FIPS_VER}.tar.gz ./

RUN set -x; \
  apk add --no-cache zlib \
  && apk add --no-cache --virtual .build-deps \
      wget \
      gcc \
      gzip \
      tar \
      libc-dev \
      ca-certificates \
      perl \
      make \
      coreutils \
      gnupg \
      linux-headers \
      zlib-dev \
      openssl \      
    && wget --quiet https://www.openssl.org/source/openssl-fips-$OPENSSL_FIPS_VER.tar.gz \
    && openssl sha1 -hmac etaonrishdlcupfm openssl-fips-$OPENSSL_FIPS_VER.tar.gz | grep $OPENSSL_FIPS_HMACSHA1 \
    && apk del openssl \
    && wget --quiet https://www.openssl.org/source/openssl-fips-$OPENSSL_FIPS_VER.tar.gz.asc \
    #&& gpg --keyserver hkp://pgp.mit.edu --recv $OPENSSL_FIPS_PGP_FINGERPRINT \
    #&& gpg --verify openssl-fips-$OPENSSL_FIPS_VER.tar.gz.asc openssl-fips-$OPENSSL_FIPS_VER.tar.gz \
    #&& echo "$OPENSSL_FIPS_HASH openssl-fips-$OPENSSL_FIPS_VER.tar.gz" | sha256sum -c - | grep OK \
    && tar -xzf openssl-fips-$OPENSSL_FIPS_VER.tar.gz \
    && cd openssl-fips-$OPENSSL_FIPS_VER \
    && ./config \
    && make \
    && make install \
    && cd .. \
    && wget --quiet https://www.openssl.org/source/openssl-$OPENSSL_VER.tar.gz \
    && wget --quiet https://www.openssl.org/source/openssl-$OPENSSL_VER.tar.gz.asc \
    && gpg --recv $OPENSSL_PGP_FINGERPRINT \
    && gpg --verify openssl-$OPENSSL_VER.tar.gz.asc \
    && echo "$OPENSSL_HASH openssl-$OPENSSL_VER.tar.gz" | sha256sum -c - | grep OK \
    && tar -xzf openssl-$OPENSSL_VER.tar.gz \
    && cd openssl-$OPENSSL_VER \
    && perl ./Configure linux-x86_64 --prefix=/usr \
                                     --libdir=lib \
                                     --openssldir=/etc/ssl \
                                     fips shared zlib enable-montasm enable-md2 enable-ec_nistp_64_gcc_128 \
                                     -DOPENSSL_NO_BUF_FREELISTS \
                                     -Wa,--noexecstack enable-ssl2 \
    && make \
    && make  INSTALL_PREFIX=/tmp/root install_sw \
  && cd \
  && rm -rf /tmp/build /usr/local/ssl \
  && apk del .build-deps

FROM alpine:3.9
COPY --from=build /tmp/root /