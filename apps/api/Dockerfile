FROM node:22-slim

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

WORKDIR /app

# Install basic dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Enable pnpm
RUN corepack enable

# Copy application code
COPY . .

# Install dependencies and build
RUN pnpm install --frozen-lockfile && \
    pnpm run build

# Start the server by default
EXPOSE 8080

# Prepare entrypoint
RUN chmod +x ./docker-entrypoint.sh && \
    sed -i 's/\r$//' ./docker-entrypoint.sh

ENTRYPOINT ["./docker-entrypoint.sh"]
CMD ["pnpm", "run", "start:production"]
