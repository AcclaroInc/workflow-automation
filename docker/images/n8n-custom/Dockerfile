ARG NODE_VERSION=22.21.1

# 1. Create an image to build n8n
FROM n8nio/base:${NODE_VERSION} AS builder

# Build the application from source
WORKDIR /src
COPY . /src
ENV CI=true
RUN pnpm install --frozen-lockfile
RUN pnpm build
# Deploy the `n8n` package into /compiled
RUN mkdir /compiled
ENV NODE_FUNCTION_ALLOW_BUILTIN=fs
ENV NODE_FUNCTION_ALLOW_EXTERNAL=xml2js
RUN NODE_ENV=production pnpm --filter=n8n --prod --legacy --no-optional deploy /compiled

# 2. Start with a new clean image with just the code that is needed to run n8n
FROM alpine:latest AS alpine
FROM n8nio/base:${NODE_VERSION}
ENV NODE_ENV=production

ARG N8N_RELEASE_TYPE=stable
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}

WORKDIR /home/node
COPY --from=builder /compiled /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /
COPY --from=alpine /sbin/apk /sbin/apk
COPY --from=alpine /usr/lib/libapk.so* /usr/lib/

RUN \
	cd /usr/local/lib/node_modules/n8n && \
	npm rebuild sqlite3 && \
	cd - && \
	ln -s /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
	mkdir .n8n && \
	chown node:node .n8n

ENV SHELL /bin/sh
RUN apk update && apk add --no-cache openssl wget zip unzip && rm -rf /var/cache/apk/*
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
