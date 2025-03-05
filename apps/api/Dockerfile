FROM node:22-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN corepack enable
WORKDIR /app

# Copy package files first for better caching
COPY package.json pnpm-lock.yaml ./
COPY apps/api/package.json ./apps/api/

FROM base AS prod-deps
# Install production dependencies (without using BuildKit cache mounts)
RUN pnpm install --prod --frozen-lockfile

FROM base AS build
# Install all dependencies and build
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm run build

# Install Go
FROM golang:1.24 AS go-base
WORKDIR /app
COPY sharedLibs/go-html-to-md ./sharedLibs/go-html-to-md

# Install Go dependencies and build parser lib
RUN cd ./sharedLibs/go-html-to-md && \
    go mod tidy && \
    go build -o html-to-markdown.so -buildmode=c-shared html-to-markdown.go && \
    chmod +x html-to-markdown.so

# Install Rust
FROM rust:1-slim AS rust-base
WORKDIR /app
COPY sharedLibs/html-transformer ./sharedLibs/html-transformer

# Install Rust dependencies and build transformer lib
RUN cd ./sharedLibs/html-transformer && \
    cargo build --release && \
    chmod +x target/release/libhtml_transformer.so

FROM base
COPY --from=build /app/dist ./dist
COPY --from=prod-deps /app/node_modules ./node_modules
COPY --from=go-base /app/sharedLibs/go-html-to-md/html-to-markdown.so ./sharedLibs/go-html-to-md/html-to-markdown.so
COPY --from=rust-base /app/sharedLibs/html-transformer/target/release/libhtml_transformer.so ./sharedLibs/html-transformer/target/release/libhtml_transformer.so
COPY . .

# Start the server by default, this can be overwritten at runtime
EXPOSE 8080

# Make sure the entrypoint script has the correct line endings
RUN chmod +x ./docker-entrypoint.sh && \
    sed -i 's/\r$//' ./docker-entrypoint.sh

ENTRYPOINT ["./docker-entrypoint.sh"]
