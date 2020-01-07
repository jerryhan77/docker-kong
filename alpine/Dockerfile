FROM alpine:latest
LABEL maintainer="Kong Core Team <team-core@konghq.com>"

ENV KONG_VERSION 1.4.2
ENV KONG_SHA256 edf917d956d697abb70f5f3f630d420ee699c6428bf953221cd8548eda490dcf

RUN adduser -Su 1337 -g root kong  \
	&& mkdir -p "/usr/local/kong" \
	&& apk add --no-cache --virtual .build-deps wget tar ca-certificates \
	&& apk add --no-cache git libgcc openssl pcre perl tzdata curl libcap su-exec unzip \
	&& wget -O kong.tar.gz "https://bintray.com/kong/kong-alpine-tar/download_file?file_path=kong-$KONG_VERSION.amd64.apk.tar.gz" \
	&& echo "$KONG_SHA256 *kong.tar.gz" | sha256sum -c - \
	&& tar -xzf kong.tar.gz -C /tmp \
	&& rm -f kong.tar.gz \
	&& cp -R /tmp/usr / \
	&& rm -rf /tmp/usr \
	&& cp -R /tmp/etc / \
	&& rm -rf /tmp/etc \
	&& luarocks install kong-oidc \
	&& luarocks install kong-oidc-auth \
	&& luarocks install kong-plugin-jwt-keycloak \
	&& apk del .build-deps \
	&& chown -R kong /usr/local/kong \
	&& chgrp -R 0 /usr/local/kong \
	&& chmod -R g=u /usr/local/kong

# for transparent proxy usage enable setcup for kong
RUN chown kong /usr/local/openresty/nginx/sbin/ \
	&& cp /usr/local/openresty/nginx/sbin/nginx /usr/local/openresty/nginx/sbin/nginx-transparent \
	&& cp /usr/local/openresty/nginx/sbin/nginx /usr/local/openresty/nginx/sbin/nginx-non-transparent \
	&& rm /usr/local/openresty/nginx/sbin/nginx \
    && setcap cap_net_raw=+ep /usr/local/openresty/nginx/sbin/nginx-transparent

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444

STOPSIGNAL SIGTERM

CMD ["kong", "docker-start"]
