FROM python:3.11-slim

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    ENV_MODE="production" \
    PYTHONPATH=/app

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user and set up directories
RUN useradd -m -u 1000 appuser && \
    mkdir -p /app/logs && \
    chown -R appuser:appuser /app

# Install Python dependencies
COPY --chown=appuser:appuser requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Switch to non-root user
USER appuser

# Copy application code
COPY --chown=appuser:appuser . .

# Expose the port the app runs on
EXPOSE 8000

# Calculate optimal worker count based on 16 vCPUs
# Using (2*CPU)+1 formula for CPU-bound applications
ENV WORKERS=33
ENV THREADS=2
ENV WORKER_CONNECTIONS=2000

EXPOSE 8000

# Gunicorn configuration
CMD ["sh", "-c", "gunicorn api:app \
     --workers $WORKERS \
     --worker-class uvicorn.workers.UvicornWorker \
     --bind 0.0.0.0:8000 \
     --timeout 1800 \
     --graceful-timeout 600 \
     --keep-alive 1800 \
     --max-requests 0 \
     --max-requests-jitter 0 \
     --forwarded-allow-ips '*' \
     --worker-connections $WORKER_CONNECTIONS \
     --worker-tmp-dir /dev/shm \
     --preload \
     --log-level info \
     --access-logfile - \
     --error-logfile - \
     --capture-output \
     --enable-stdio-inheritance \
     --threads $THREADS"]

# 인증서 설정을 위해 루트 권한 획득
USER root

# 루트 인증서 복사
COPY SK_SSL.crt /usr/local/share/ca-certificates/SK_SSL.crt

# 시스템 CA 인증서 업데이트
RUN update-ca-certificates

# certifi가 참조하는 경로를 시스템 CA로 링크
RUN ln -sf /etc/ssl/certs/ca-certificates.crt $(python3 -m certifi)

# appuser로 변경
USER appuser

# 환경 변수도 같이 설정
ENV REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
ENV CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt