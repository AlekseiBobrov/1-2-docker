# Stage 1: Build Vue.js app
FROM node:16-alpine AS builder

ARG VUE_APP_API_URL
ENV VUE_APP_API_URL=$VUE_APP_API_URL

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

RUN npm run build

# Stage 2: Build runtime image with Nginx
FROM nginx:1.21-alpine

RUN apk add --no-cache curl

RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/conf.d/

COPY --from=builder /app/dist /usr/share/nginx/html

RUN adduser -D -u 1001 -h /usr/share/nginx/html nginxuser

RUN mkdir -p /var/cache/nginx /var/log/nginx /var/run && \
    chown -R nginxuser:nginxuser /var/cache/nginx && \
    chown -R nginxuser:nginxuser /var/log/nginx && \
    chown -R nginxuser:nginxuser /var/run && \
    chown -R nginxuser:nginxuser /usr/share/nginx/html && \
    touch /var/run/nginx.pid && \
    chown nginxuser:nginxuser /var/run/nginx.pid

USER nginxuser

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -fsS http://localhost:80/ || exit 1

CMD ["nginx", "-g", "daemon off;"]