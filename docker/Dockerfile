FROM python:3.10-slim

WORKDIR /app

# Latest releases available at https://github.com/aptible/supercronic/releases
ARG TARGETARCH
ENV SUPERCRONIC_VERSION=v0.2.39

RUN set -ex && \
    apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    case ${TARGETARCH} in \
    amd64) \
    export SUPERCRONIC_URL=https://github.com/aptible/supercronic/releases/download/${SUPERCRONIC_VERSION}/supercronic-linux-amd64; \
    export SUPERCRONIC_SHA1SUM=c98bbf82c5f648aaac8708c182cc83046fe48423; \
    export SUPERCRONIC=supercronic-linux-amd64; \
    ;; \
    arm64) \
    export SUPERCRONIC_URL=https://github.com/aptible/supercronic/releases/download/${SUPERCRONIC_VERSION}/supercronic-linux-arm64; \
    export SUPERCRONIC_SHA1SUM=5ef4ccc3d43f12d0f6c3763758bc37cc4e5af76e; \
    export SUPERCRONIC=supercronic-linux-arm64; \
    ;; \
    *) \
    echo "Unsupported architecture: ${TARGETARCH}"; \
    exit 1; \
    ;; \
    esac && \
    echo "Downloading supercronic for ${TARGETARCH} from ${SUPERCRONIC_URL}" && \
    # 重试机制：最多3次，每次超时30秒
    for i in 1 2 3; do \
        echo "Download attempt $i/3"; \
        if curl -fsSL --connect-timeout 30 --max-time 60 -o "$SUPERCRONIC" "$SUPERCRONIC_URL"; then \
            echo "Download successful"; \
            break; \
        else \
            echo "Download attempt $i failed"; \
            if [ $i -eq 3 ]; then \
                echo "All download attempts failed"; \
                exit 1; \
            fi; \
            sleep 2; \
        fi; \
    done && \
    echo "${SUPERCRONIC_SHA1SUM}  ${SUPERCRONIC}" | sha1sum -c - && \
    chmod +x "$SUPERCRONIC" && \
    mv "$SUPERCRONIC" "/usr/local/bin/${SUPERCRONIC}" && \
    ln -s "/usr/local/bin/${SUPERCRONIC}" /usr/local/bin/supercronic && \
    supercronic -version && \
    apt-get remove -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ========== 关键修改：所有COPY路径添加../，指向项目根目录 ==========
# 原：COPY requirements.txt . → 改后：COPY ../requirements.txt .
COPY ../requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 原：COPY docker/manage.py . → 改后：COPY manage.py .（因为manage.py和Dockerfile同目录）
COPY manage.py .
# 原：COPY trendradar/ ./trendradar/ → 改后：COPY ../trendradar/ ./trendradar/
COPY ../trendradar/ ./trendradar/

# 原：COPY docker/entrypoint.sh /entrypoint.sh.tmp → 改后：COPY entrypoint.sh /entrypoint.sh.tmp（同目录）
COPY entrypoint.sh /entrypoint.sh.tmp
# ========== 关键修改结束 ==========

RUN sed -i 's/\r$//' /entrypoint.sh.tmp && \
    mv /entrypoint.sh.tmp /entrypoint.sh && \
    chmod +x /entrypoint.sh && \
    chmod +x manage.py && \
    mkdir -p /app/config /app/output

ENV PYTHONUNBUFFERED=1 \
    CONFIG_PATH=/app/config/config.yaml \
    FREQUENCY_WORDS_PATH=/app/config/frequency_words.txt

ENTRYPOINT ["/entrypoint.sh"]
