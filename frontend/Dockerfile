FROM alpine AS builder
WORKDIR /app
COPY index.html app.js ./

FROM nginx:alpine
COPY default.conf /etc/nginx/conf.d/default.conf

COPY --from=builder /app/index.html /usr/share/nginx/html/
COPY --from=builder /app/app.js /usr/share/nginx/html/

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]