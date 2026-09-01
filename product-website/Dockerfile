# The product site: built with Vite, served by nginx.
#
# Two stages so the built assets ship without Node or node_modules — the
# runtime image is a few megabytes of static files and a web server.

FROM node:22-alpine AS builder
WORKDIR /app

# Dependencies first, so a source-only change does not reinstall them.
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:1.27-alpine AS runtime

# A single-page app has one real file; every path has to fall back to it or a
# refresh on any route below / returns 404 from nginx rather than the app.
RUN printf 'server {\n\
  listen 80;\n\
  root /usr/share/nginx/html;\n\
  index index.html;\n\
  location / {\n\
    try_files $uri $uri/ /index.html;\n\
  }\n\
}\n' > /etc/nginx/conf.d/default.conf

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

# wget is in the busybox base, which is what the fleet health probe uses.
HEALTHCHECK --interval=15s --timeout=5s --start-period=5s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1/ || exit 1
