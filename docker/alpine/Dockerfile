FROM alpine AS builder
LABEL maintainer="kev <noreply@datageek.info>, Sah <contact@leesah.name>, vndroid <waveworkshop@outlook.com>"

COPY . /tmp/repo
RUN set -x \
 # Build environment setup
 && apk add --no-cache --virtual .build-deps \
      build-base \
      cmake \
      c-ares-dev \
      libcap \
      libev-dev \
      libsodium-dev \
      linux-headers \
      mbedtls-dev \
      pcre2-dev \
 # Build & install
 && cd /tmp/repo \
 && mkdir -p build && cd build \
 && cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_TESTING=OFF -DWITH_STATIC=OFF -DCMAKE_BUILD_TYPE=Release \
 && make -j$(getconf _NPROCESSORS_ONLN) \
 && make install \
 && cd /usr/local/bin \
 && ls /usr/local/bin/ss-* | xargs -n1 setcap cap_net_bind_service+ep \
 && for f in ss-local ss-manager ss-redir ss-server ss-tunnel; do \
      if file "/usr/local/bin/$f" | grep -q 'ELF'; then \
        strip "/usr/local/bin/$f"; \
      fi \
    done \
 && apk del .build-deps \
 # Runtime dependencies setup
 && apk add --no-cache \
      ca-certificates \
      rng-tools \
      tzdata \
      $(scanelf --needed --nobanner /usr/local/bin/ss-* \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u) \
 && rm -rf /tmp/repo

FROM alpine

ENV SERVER_ADDR=0.0.0.0
ENV SERVER_PORT=8388
ENV PASSWORD=
ENV METHOD=aes-256-gcm
ENV TIMEOUT=300
ENV DNS_ADDRS="8.8.8.8,8.8.4.4"
ENV TZ=UTC
ENV ARGS=

COPY --from=builder \
    /usr/local/bin/ss-server \
    /usr/local/bin/

RUN set -x \
 # Runtime dependencies setup
 && apk add --no-cache \
      ca-certificates \
      rng-tools \
      tzdata \
      $(scanelf --needed --nobanner /usr/local/bin/ss-* \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u) \
 && rm -rf /var/cache/apk/* \
 && rm -rf /tmp/*

COPY ./docker/alpine/entrypoint.sh /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8388

STOPSIGNAL SIGINT

CMD ["ss-server"]
