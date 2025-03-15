ARG NODE_VERSION=20
FROM n8nio/base:${NODE_VERSION}

ARG N8N_VERSION
RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

LABEL org.opencontainers.image.title="n8n"
LABEL org.opencontainers.image.description="Workflow Automation Tool"
LABEL org.opencontainers.image.source="https://github.com/n8n-io/n8n"
LABEL org.opencontainers.image.url="https://n8n.io"
LABEL org.opencontainers.image.version=${N8N_VERSION}

# Set environment variables
ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable
ENV N8N_PORT=5678
ENV PORT=5678

# Install n8n
RUN set -eux; \
    npm install -g --omit=dev n8n@${N8N_VERSION} --ignore-scripts && \
    npm rebuild --prefix=/usr/local/lib/node_modules/n8n sqlite3 && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/chat && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/design-system && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/node_modules && \
    find /usr/local/lib/node_modules/n8n -type f -name "*.ts" -o -name "*.js.map" -o -name "*.vue" | xargs rm -f && \
    rm -rf /root/.npm

# Setup the Task Runner Launcher
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.1.1
COPY n8n-task-runners.json /etc/n8n-task-runners.json

# Download, verify, then extract the launcher binary
RUN \
    if [[ "$TARGETPLATFORM" = "linux/amd64" ]]; then export ARCH_NAME="amd64"; \
    elif [[ "$TARGETPLATFORM" = "linux/arm64" ]]; then export ARCH_NAME="arm64"; fi; \
    mkdir /launcher-temp && \
    cd /launcher-temp && \
    wget https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz && \
    wget https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256 && \
    # The .sha256 does not contain the filename --> Form the correct checksum file
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256 && \
    sha256sum -c checksum.sha256 && \
    tar xvf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz --directory=/usr/local/bin && \
    cd - && \
    rm -r /launcher-temp

# Install Puppeteer & Chromium
RUN npm install -g n8n-nodes-puppeteer
RUN apt-get update && apt-get install -y chromium

COPY docker-entrypoint.sh /

# Set up working directory & permissions
RUN \
    mkdir -p /home/node/.n8n && \
    chown -R node:node /home/node/.n8n

# Expose port 5678 for n8n
EXPOSE 5678

# Switch to non-root user
USER node

# Start n8n service
CMD ["n8n"]
