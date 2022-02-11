FROM nginx:1.21

ARG nginx_gid=1001
ARG nginx_uid=1001

ADD app/ /app
WORKDIR /app

COPY ./conf/nginx.conf /etc/nginx/nginx.conf
COPY ./conf/default.conf /etc/nginx/conf.d/default.conf

RUN usermod -u $nginx_uid -o nginx           && \
    groupmod -g $nginx_gid -o nginx          && \
    chown -R nginx:nginx /app                && \
    chmod -R 755 /app                        && \
    chown -R nginx:nginx /var/cache/nginx    && \
    chown -R nginx:nginx /var/log/nginx      && \
    chown -R nginx:nginx /etc/nginx/conf.d

RUN touch /var/run/nginx.pid                 && \
    chown -R nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 8000
