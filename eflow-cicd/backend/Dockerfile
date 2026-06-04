# ─────────────────────────────────────────
# E-Flow Microservice - Dockerfile
# Node.js Backend (userservice / authservice / courseservice etc.)
# ─────────────────────────────────────────

# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies first (cache layer)
COPY package*.json ./
RUN npm install --production

# Copy source code
COPY . .

# ─────────────────────────────────────────
# Stage 2: Production Image
FROM node:18-alpine

WORKDIR /app

# Copy only production dependencies + built code
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app .

# Security: non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# App port (change if your service uses different port)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

# Start command
CMD ["node", "server.js"]
