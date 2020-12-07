FROM nginx:1.18.0-alpine

LABEL maintainer="sergey.kudelin@gmail.com"

COPY default.conf /etc/nginx/conf.d/

RUN rm -f /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh

## add permissions
RUN addgroup -g 1001 web \
    && adduser -u 1001 -G web -s /bin/sh -D web

RUN chown -R web:web /run && \
    chown -R web:web /var/cache/nginx && \
    chown -R nobody.nobody /var/log/nginx

## switch to non-root user
USER web

CMD ["nginx", "-g", "daemon off;"]