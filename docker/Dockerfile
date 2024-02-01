# STEP2: BUILD THE FRONTEND
FROM node:18-alpine AS builder_frontend
WORKDIR /home/
COPY . ./
RUN apk add make git gzip brotli && \
    npm install --legacy-peer-deps && \
    make build_frontend && \
    cd public && make compress

# STEP3: BUILD THE BACKEND
FROM golang:1.19-bookworm AS builder_backend
WORKDIR /home/
COPY --from=builder_frontend /home/ ./
RUN apt-get update > /dev/null && \
    apt-get install -y libvips-dev curl make > /dev/null 2>&1 && \
    apt-get install -y libjpeg-dev libtiff-dev libpng-dev libwebp-dev libraw-dev libheif-dev libgif-dev && \
    make build_init && \
    make build_backend && \
    mkdir -p ./dist/data/state/config/ && \
    cp config/config.json ./dist/data/state/config/config.json

# STEP4: Create the prod image from the build
FROM machines/filestash:latest
ARG TARGETPLATFORM
USER root
RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
    apt update && \
    apt install -y --no-install-recommends libgomp1 libheif-dev liblcms2-dev libtiff-dev && \
    rm -rf /var/lib/apt/lists/*; \
    fi
COPY --from=mwader/static-ffmpeg:latest /ffmpeg /ffprobe /usr/local/bin/
COPY --from=builder_backend --chown=filestash:filestash /home/dist/ /app/
USER filestash
