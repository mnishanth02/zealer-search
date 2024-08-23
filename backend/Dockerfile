FROM node:20-alpine AS base

FROM base AS builder
WORKDIR /app

RUN apk add --no-cache gcompat
RUN npm install -g pnpm
COPY pnpm-lock.yaml ./
RUN pnpm fetch

COPY . .

RUN pnpm install
RUN pnpm run build


FROM base AS runner
WORKDIR /app

RUN apk add --no-cache libc6-compat
RUN npm install -g pnpm

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json
COPY --from=builder --chown=hono:nodejs /app/pnpm-lock.yaml /app/pnpm-lock.yaml

RUN pnpm install --prod

USER hono
EXPOSE 3002

CMD ["node", "/app/dist/index.js"]