# apps/docs/Dockerfile
FROM node:24-alpine AS base

# ---------------------------------------------------------
# Stage 1: Prune workspace to only the 'docs' app
# ---------------------------------------------------------
FROM base AS builder
RUN apk add --no-cache libc6-compat
WORKDIR /app
RUN npm install -g turbo
COPY . .

RUN turbo prune docs --docker

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

RUN pnpm turbo run build --filter=docs...

# ---------------------------------------------------------
# Stage 3: Production Runner
# ---------------------------------------------------------
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs


COPY --from=installer /app/apps/docs/public ./public
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/static ./apps/docs/.next/static

USER nextjs

EXPOSE 3001
ENV PORT=3001
CMD ["node", "apps/docs/server.js"]