FROM node:20-alpine AS builder
WORKDIR /app

# Install dependencies (prefer lockfile if present)
COPY package.json package-lock.json* ./
RUN npm ci --silent --no-audit --no-fund || npm install --silent

# Copy source and build
COPY . .
RUN npm run build -- --configuration production || npm run build

FROM nginx:stable-alpine AS runner

# Remove default nginx content and copy build artifacts.
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
