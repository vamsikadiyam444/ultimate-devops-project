# syntax=docker/dockerfile:1

# --- BUILD STAGE ---
ARG RUST_VERSION=1.88.0
ARG APP_NAME=shipping

FROM rust:${RUST_VERSION}-alpine AS build
ARG APP_NAME

# Set working directory
WORKDIR /app

# Install required packages
RUN apk add --no-cache clang lld musl-dev git protobuf

# Copy full project into the container for build
COPY . .

# Cache Rust dependencies and build the application
RUN --mount=type=cache,target=/app/target/ \
    --mount=type=cache,target=/usr/local/cargo/git/db \
    --mount=type=cache,target=/usr/local/cargo/registry/ \
    cargo build --locked --release && \
    cp ./target/release/${APP_NAME} /bin/server

# --- FINAL RUNTIME STAGE ---
FROM alpine:3.18 AS final

# Create non-root user
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

USER appuser

# Copy the built binary from build stage
COPY --from=build /bin/server /bin/

# Expose the port your application listens on
EXPOSE 7077

# Default command to run
CMD ["/bin/server"]


