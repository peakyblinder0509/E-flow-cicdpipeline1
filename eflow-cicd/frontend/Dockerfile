# ─────────────────────────────────────────
# E-Flow Frontend - Dockerfile
# React (Vite) — Multi-stage build
# ─────────────────────────────────────────

# Stage 1: Build React App
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source and build
COPY . .
RUN npm run build
# Output: /app/dist  (Vite)
# Output: /app/build (CRA - change below accordingly)

# ─────────────────────────────────────────
# Stage 2: Serve via Nginx
FROM nginx:alpine

# Copy built files to nginx html folder
COPY --from=builder /app/dist /usr/share/nginx/html

# Custom nginx config for React Router support
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
