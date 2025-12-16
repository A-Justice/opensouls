# Soul Engine Documentation
FROM oven/bun:1.2 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
COPY packages/beta-docs/package.json ./packages/beta-docs/
RUN bun install --frozen-lockfile

# Build stage
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

WORKDIR /app/packages/beta-docs

ENV NEXT_TELEMETRY_DISABLED=1
RUN bun run build

# Production stage
FROM base AS runner
WORKDIR /app/packages/beta-docs

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3001

COPY --from=builder /app/packages/beta-docs/.next ./.next
COPY --from=builder /app/packages/beta-docs/public ./public
COPY --from=builder /app/packages/beta-docs/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3001

CMD ["bun", "run", "start"]

