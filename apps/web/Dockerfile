FROM node:24-alpine AS base

# ---------------------------------------------------------
# Stage 1: Prune workspace to only the 'web' app
# ---------------------------------------------------------
FROM base AS builder
RUN apk add --no-cache libc6-compat
WORKDIR /app
RUN npm install -g turbo
COPY . .
RUN turbo prune web --docker

# ---------------------------------------------------------
# Stage 2: Install and Build
# ---------------------------------------------------------
FROM base AS installer
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install pnpm natively
RUN corepack enable pnpm

# Install dependencies
COPY --from=builder /app/out/json/ .
RUN pnpm install

# Build the project
COPY --from=builder /app/out/full/ .
RUN pnpm turbo run build --filter=web...

# ---------------------------------------------------------
# Stage 3: Production Runner
# ---------------------------------------------------------
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=installer /app/apps/web/public ./public
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000
CMD ["node", "apps/web/server.js"]
