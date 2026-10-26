# ==========================================
# Stage 1: Builder (Heavy dependencies)
# ==========================================
FROM python:3.11-slim AS builder

# Suppress debconf warnings
ARG DEBIAN_FRONTEND=noninteractive

WORKDIR /app
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# ==========================================
# Stage 2: Runtime (Lightweight)
# ==========================================
FROM python:3.11-slim

# Suppress warnings
ARG DEBIAN_FRONTEND=noninteractive
ENV PIP_ROOT_USER_ACTION=ignore
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /usr/src/app

RUN apt-get update && apt-get install -y --no-install-recommends \
    libgl1 \
    libglib2.0-0 \
    tesseract-ocr \
    tesseract-ocr-eng \
    tesseract-ocr-tel \
    fonts-dejavu \
    fonts-noto-core \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .

# Install wheels and immediately remove the wheels folder to save space
RUN pip install --no-cache /wheels/* && rm -rf /wheels

COPY . .

# Non-root user for security
RUN useradd -m appuser && chown -R appuser:appuser /usr/src/app
USER appuser

EXPOSE 3000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:3000", "app:app", "--timeout", "120"]


docker rm -f app && \
docker build --no-cache -t srinuas/pdfimg:1 . && \
docker run -itd --name app -p 333:3000 --env-file .env srinuas/pdfimg:1


docker exec -it app ls -la /usr/src/app


