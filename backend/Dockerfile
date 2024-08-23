FROM node:20-alpine AS base

FROM base AS builder
ARG SEARXNG_API_URL
ENV SEARXNG_API_URL=${SEARXNG_API_URL}

WORKDIR /app

RUN apk add --no-cache gcompat
RUN npm install -g pnpm
COPY pnpm-lock.yaml ./
RUN pnpm fetch

COPY . .

RUN pnpm install
RUN pnpm run build


FROM base AS runner
ARG SEARXNG_API_URL
ENV SEARXNG_API_URL=${SEARXNG_API_URL}
WORKDIR /app

RUN apk add --no-cache libc6-compat
RUN npm install -g pnpm

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json
COPY --from=builder --chown=hono:nodejs /app/pnpm-lock.yaml /app/pnpm-lock.yaml

# RUN sed -i "s|SEARXNG = \".*\"|SEARXNG = \"${SEARXNG_API_URL}\"|g" /app/config.toml

RUN pnpm install --prod

USER hono
EXPOSE 3001

CMD ["node", "/app/dist/index.js"]