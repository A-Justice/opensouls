# Soul Engine Cloud - Backend Server
FROM oven/bun:1.2 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
COPY packages/soul-engine-cloud/package.json ./packages/soul-engine-cloud/
COPY packages/core/package.json ./packages/core/
COPY packages/engine/package.json ./packages/engine/
COPY packages/soul/package.json ./packages/soul/
COPY packages/pipeline/package.json ./packages/pipeline/
RUN bun install --frozen-lockfile

# Build stage
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/packages/soul-engine-cloud/node_modules ./packages/soul-engine-cloud/node_modules
COPY . .

# Generate Prisma client
WORKDIR /app/packages/soul-engine-cloud
RUN bunx prisma generate

# Production stage
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV DEBUG_SERVER_PORT=4000

# Copy necessary files
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/packages ./packages
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/tsconfig.json ./tsconfig.json

# Create data directories
RUN mkdir -p /app/data/pglite /app/data/souls

WORKDIR /app/packages/soul-engine-cloud

# Expose ports
EXPOSE 4000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:4000/health || exit 1

# Start the server
CMD ["bun", "run", "dev"]

