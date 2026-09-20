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

# Default values are intentionally safe placeholders so startup validation can pass
# in a container. Override these at runtime with real secrets and URLs in Docker
# or your hosting platform.
ENV NODE_ENV=production
ENV PORT=3000
ENV API_DOCS_ENABLED=false
ENV DATABASE_PROVIDER=mysql
ENV DATABASE_URL=mysql://user:password@localhost:3306/invinceible_core_hms_v2
ENV JWT_SECRET=development-jwt-secret-that-is-long-enough-123456
ENV JWT_EXPIRES_IN=1d
ENV LAB_SIGNING_KEY=development-lab-signing-key-that-is-long-enough-123456
ENV FRONTEND_URL=http://localhost:3001
ENV FRONTEND_ORIGINS=http://localhost:3001,http://localhost:3000
ENV AUTH_COOKIE_SAME_SITE=lax
ENV AUTH_COOKIE_SECURE=false
ENV BODY_LIMIT=4mb
ENV QUEUE_ENABLED=true
ENV WORKER_MODE=false

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

CMD ["node", "dist/src/main.js"]