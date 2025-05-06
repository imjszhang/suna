FROM python:3.11-slim

# 添加构建参数声明
ARG ENV_MODE
ARG REDIS_HOST
ARG REDIS_PORT
ARG REDIS_PASSWORD
ARG REDIS_SSL
ARG NEXT_PUBLIC_URL
ARG SUPABASE_URL
ARG SUPABASE_ANON_KEY
ARG SUPABASE_SERVICE_ROLE_KEY
ARG DAYTONA_API_KEY
ARG DAYTONA_SERVER_URL
ARG DAYTONA_TARGET
ARG ANTHROPIC_API_KEY
ARG OPENAI_API_KEY
ARG TAVILY_API_KEY
ARG FIRECRAWL_API_KEY
ARG RAPID_API_KEY
ARG API_DOMAIN

# 设置环境变量
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH=/app

# 设置从构建参数传递的环境变量
ENV ENV_MODE=${ENV_MODE:-"development"} \
    REDIS_HOST=${REDIS_HOST} \
    REDIS_PORT=${REDIS_PORT} \
    REDIS_PASSWORD=${REDIS_PASSWORD} \
    REDIS_SSL=${REDIS_SSL} \
    NEXT_PUBLIC_URL=${NEXT_PUBLIC_URL} \
    SUPABASE_URL=${SUPABASE_URL} \
    SUPABASE_ANON_KEY=${SUPABASE_ANON_KEY} \
    SUPABASE_SERVICE_ROLE_KEY=${SUPABASE_SERVICE_ROLE_KEY} \
    DAYTONA_API_KEY=${DAYTONA_API_KEY} \
    DAYTONA_SERVER_URL=${DAYTONA_SERVER_URL} \
    DAYTONA_TARGET=${DAYTONA_TARGET} \
    ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY} \
    OPENAI_API_KEY=${OPENAI_API_KEY} \
    TAVILY_API_KEY=${TAVILY_API_KEY} \
    FIRECRAWL_API_KEY=${FIRECRAWL_API_KEY} \
    RAPID_API_KEY=${RAPID_API_KEY} \
    API_DOMAIN=${API_DOMAIN}

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

# Gunicorn configuration
CMD ["sh", "-c", "gunicorn api:app \
     --workers $WORKERS \
     --worker-class uvicorn.workers.UvicornWorker \
     --bind 0.0.0.0:8000 \
     --timeout 600 \
     --graceful-timeout 300 \
     --keep-alive 250 \
     --max-requests 2000 \
     --max-requests-jitter 400 \
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
