FROM docker.io/bitnami/minideb:bookworm

ARG TARGETARCH
ARG PHPBB_VERSION="3.3.15"           # ← change this to pin a different phpBB release
ARG PHPBB_LIBRARY_VERSION="3.3"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-12" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      acl ca-certificates curl jq \
      libaudit1 libbrotli1 libbsd0 libbz2-1.0 libcap-ng0 libcom-err2 libcrypt1 \
      libcurl4 libexpat1 libffi8 libfftw3-double3 libfontconfig1 libfreetype6 \
      libgcc-s1 libgcrypt20 libglib2.0-0 libgmp10 libgnutls30 libgomp1 libgpg-error0 \
      libgssapi-krb5-2 libhashkit2 libhogweed6 libicu72 libidn2-0 libjpeg62-turbo \
      libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 liblcms2-2 libldap-2.5-0 \
      liblqr-1-0 libltdl7 liblzma5 libmagickcore-6.q16-6 libmagickwand-6.q16-6 \
      libmd0 libmemcached11 libncurses6 libnettle8 libnghttp2-14 libonig5 \
      libp11-kit0 libpam0g libpcre2-8-0 libpcre3 libpng16-16 libpq5 libpsl5 \
      libreadline8 librtmp1 libsasl2-2 libsodium23 libsqlite3-0 libssh2-1 \
      libssl3 libstdc++6 libsybdb5 libtasn1-6 libtidy5deb1 libtinfo6 \
      libunistring2 libuuid1 libwebp7 libx11-6 libxau6 libxcb1 libxdmcp6 \
      libxext6 libxml2 libxslt1.1 libzip4 libzstd1 openssl procps zlib1g unzip zip bzip2\
    && rm -rf /var/lib/apt/lists/ /var/cache/apt/archives*

# 2) Download & unpack the Bitnami components
RUN mkdir -p /tmp/bitnami/pkg/cache/ \
 && cd /tmp/bitnami/pkg/cache/ \
 && COMPONENTS=( \
      "render-template-1.0.7-16-linux-${OS_ARCH}-debian-12" \
      "php-8.2.28-8-linux-${OS_ARCH}-debian-12" \
      "apache-2.4.63-2-linux-${OS_ARCH}-debian-12" \
      "mysql-client-11.4.6-0-linux-${OS_ARCH}-debian-12" \
      "libphp-8.2.28-0-linux-${OS_ARCH}-debian-12" \
    ) \
 && for COMPONENT in "${COMPONENTS[@]}"; do \
      curl -SsLf "https://downloads.bitnami.com/files/stacksmith/${COMPONENT}.tar.gz" -O; \
      curl -SsLf "https://downloads.bitnami.com/files/stacksmith/${COMPONENT}.tar.gz.sha256" -O; \
      sha256sum -c "${COMPONENT}.tar.gz.sha256"; \
      tar -xzf "${COMPONENT}.tar.gz" -C /opt/bitnami --strip-components=2 --no-same-owner --wildcards '*/files'; \
      rm -f "${COMPONENT}.tar.gz"{,.sha256}; \
    done

# 3) Now pull phpBB from the official phpBB site
RUN curl -fsSL "https://download.phpbb.com/pub/release/${PHPBB_LIBRARY_VERSION}/${PHPBB_VERSION}/phpBB-${PHPBB_VERSION}.tar.bz2" \
      -o /tmp/phpbb.tar.bz2 \
 && mkdir -p /opt/bitnami/phpbb \
 && tar -xjf /tmp/phpbb.tar.bz2 -C /opt/bitnami/phpbb --strip-components=1 \
 && rm /tmp/phpbb.tar.bz2

# 4) Cleanup & permissions
RUN apt-get update && apt-get upgrade -y \
 && apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives \
 && chmod g+rwX /opt/bitnami \
 && find / -perm /6000 -type f -exec chmod a-s {} \; || true

COPY rootfs /

RUN find /opt/bitnami/scripts -type f -name '*.sh' \
    -exec sed -i 's/\r$//' {} \; \
  && find /opt/bitnami/scripts -type f -name '*.sh' \
    -exec chmod +x {} \;
RUN find / -maxdepth 1 -type f -name '*.sh' \
    -exec sed -i 's/\r$//' {} \; \
    -exec chmod +x {} \;

RUN /opt/bitnami/scripts/mysql-client/postunpack.sh
RUN /opt/bitnami/scripts/apache/postunpack.sh
RUN /opt/bitnami/scripts/php/postunpack.sh
RUN /opt/bitnami/scripts/apache-modphp/postunpack.sh
RUN /opt/bitnami/scripts/phpbb/postunpack.sh

ENV APACHE_HTTPS_PORT_NUMBER="" \
    APACHE_HTTP_PORT_NUMBER="" \
    APP_VERSION="${PHPBB_VERSION}" \
    BITNAMI_APP_NAME="phpbb" \
    PATH="/opt/bitnami/common/bin:/opt/bitnami/php/bin:/opt/bitnami/php/sbin:/opt/bitnami/apache/bin:/opt/bitnami/mysql/bin:$PATH"

EXPOSE 8080 8443


ENTRYPOINT [ "/opt/bitnami/scripts/phpbb/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/apache/run.sh" ]