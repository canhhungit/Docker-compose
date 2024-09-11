
# Build BASE
FROM node:20.9-alpine as BASE
LABEL author="HungNC"

WORKDIR /app
COPY package.json *.lock ./
RUN apk add --no-cache git \
    && yarn --frozen-lockfile \
    && yarn cache clean

# Build Image
FROM node:20.9-alpine AS BUILD
LABEL author="HungNC"

WORKDIR /app
COPY --from=BASE /app/node_modules ./node_modules
COPY . .
RUN apk add --no-cache git curl \
    && yarn build \
    && cd .next/standalone

# Prune unnecessary files from ./node_modules, such as markdown, typescript source files, and so on
RUN curl -sf https://gobinaries.com/tj/node-prune | sh -s -- -b /usr/local/bin \
    && apk del curl \
    && node-prune

# Build production
FROM node:20.9-alpine AS PRODUCTION
LABEL author="HungNC"

WORKDIR /app

COPY --from=BUILD /app/public ./public
COPY --from=BUILD /app/next.config.mjs ./

# Set mode "standalone" in file "next.config.mjs"
COPY --from=BUILD /app/.next/standalone ./
COPY --from=BUILD /app/.next/static ./.next/static

EXPOSE 3000

CMD ["node", "server.js"]
