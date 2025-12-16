# Soul Engine UI - Next.js Frontend
FROM oven/bun:1.2 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
COPY packages/soul-engine-ui/package.json ./packages/soul-engine-ui/
COPY packages/core/package.json ./packages/core/
COPY packages/engine/package.json ./packages/engine/
COPY packages/soul/package.json ./packages/soul/
COPY packages/react/package.json ./packages/react/
RUN bun install --frozen-lockfile

# Build stage
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build the Next.js app
WORKDIR /app/packages/soul-engine-ui

ARG NEXT_PUBLIC_HOCUS_POCUS_HOST
ARG NEXT_PUBLIC_SITE_URL

ENV NEXT_PUBLIC_HOCUS_POCUS_HOST=${NEXT_PUBLIC_HOCUS_POCUS_HOST:-ws://localhost:4000}
ENV NEXT_PUBLIC_SITE_URL=${NEXT_PUBLIC_SITE_URL:-http://localhost:3000}
ENV NEXT_TELEMETRY_DISABLED=1

RUN bun run build

# Production stage
FROM base AS runner
WORKDIR /app/packages/soul-engine-ui

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/packages/soul-engine-ui/public ./public

# Set permissions for prerender cache
RUN mkdir -p .next
RUN chown nextjs:nodejs .next

# Copy built standalone application
COPY --from=builder --chown=nextjs:nodejs /app/packages/soul-engine-ui/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/packages/soul-engine-ui/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]

