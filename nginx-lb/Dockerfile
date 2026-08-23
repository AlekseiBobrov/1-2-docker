FROM nginx:1.21-alpine

RUN apk add --no-cache curl

RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/conf.d/

RUN adduser -D -u 1001 -h /etc/nginx nginxuser && \
    mkdir -p /var/cache/nginx /var/log/nginx /var/run && \
    chown -R nginxuser:nginxuser /var/cache/nginx && \
    chown -R nginxuser:nginxuser /var/log/nginx && \
    chown -R nginxuser:nginxuser /var/run && \
    chown -R nginxuser:nginxuser /etc/nginx && \
    touch /var/run/nginx.pid && \
    chown nginxuser:nginxuser /var/run/nginx.pid

USER nginxuser

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -fsS http://localhost:8081/health || exit 1

CMD ["nginx", "-g", "daemon off;"]