FROM nginx:alpine

MAINTAINER Rob Contreras <rob.contreras@ekeepergroup.co.uk>

ADD nginx.conf /etc/nginx/
COPY conf/*.conf /etc/nginx/sites-available/

RUN apk update \
    && apk upgrade \
    && apk add --no-cache bash \
    && adduser -D -H -u 1000 -s /bin/bash www-data \
    && rm /etc/nginx/conf.d/default.conf

CMD ["nginx"]

EXPOSE 80 443
