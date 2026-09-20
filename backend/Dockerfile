FROM node:22-alpine AS builder
WORKDIR /app

RUN apk add --no-cache openssl

COPY backend/package.json backend/package-lock.json ./
COPY backend/prisma ./prisma
COPY backend/prisma-postgresql ./prisma-postgresql
COPY backend/prisma.config.ts ./prisma.config.ts
COPY backend/scripts ./scripts
RUN npm ci

COPY backend/. .
RUN npm run build
RUN npm prune --omit=dev

FROM node:22-alpine AS runner
WORKDIR /app

RUN apk add --no-cache openssl

ENV NODE_ENV=production
ENV PORT=3000

RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nestjs

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/prisma ./prisma
COPY --from=builder --chown=nestjs:nodejs /app/prisma-postgresql ./prisma-postgresql
COPY --from=builder --chown=nestjs:nodejs /app/prisma.config.ts ./prisma.config.ts
COPY --from=builder --chown=nestjs:nodejs /app/scripts ./scripts

USER nestjs
EXPOSE 3000

CMD ["node", "dist/main.js"]