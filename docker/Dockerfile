ARG ALPINE_VERSION=3.19
ARG NODE_VERSION=18

# Builder: contains dependencies (node_modules) and node
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS builder
WORKDIR /app
COPY package*.json ./
RUN apk add make gcc g++ python3 linux-headers git
RUN npm ci --production --no-audit --no-optional --no-update-notifier
# Serialport needs to be rebuild for Alpine https://serialport.io/docs/9.x.x/guide-installation#alpine-linux
RUN npm rebuild --build-from-source

# Final image
FROM alpine:${ALPINE_VERSION}
WORKDIR /app
RUN apk add --no-cache tzdata eudev tini nodejs

COPY --from=builder /usr/local/bin/node /usr/local/bin/
COPY --from=builder /app/node_modules ./node_modules
COPY dist ./dist
COPY package.json LICENSE index.js data/configuration.yaml data/configuration.example.yaml ./
COPY docker/docker-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-entrypoint.sh
RUN mkdir /app/data

ARG COMMIT
RUN echo "$COMMIT" > dist/.hash

ENV NODE_ENV production

ENTRYPOINT ["docker-entrypoint.sh"]
RUN docker-entrypoint.sh /sbin/tini -- node index.js
CMD [ "/sbin/tini", "--", "node", "index.js"]
