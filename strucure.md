Successfully tagged srinuas/pdfimg:1
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
571dc79a079ca642e9091ab4b484b89d7730ea1aa22d730ebd5844ecf8fc6722
root@dc:~/srinu/pdf_edit# 
root@dc:~/srinu/pdf_edit# docker exec -it app ls -la /usr/src/app
total 372
drwxr-xr-x 5 root root   4096 Oct 26 13:09 .
drwxr-xr-x 1 root root   4096 Oct 26 13:11 ..
-rw-r--r-- 1 root root    151 Oct 26 11:17 .dockerignore
-rw-r--r-- 1 root root    318 Oct 26 13:13 .env
-rw-r--r-- 1 root root    336 Oct 26 11:47 .env.gpg
drwxr-xr-x 7 root root   4096 Oct 26 09:27 .git
-rw-r--r-- 1 root root    243 Oct 26 11:16 .gitignore
-rw-r--r-- 1 root root   1596 Oct 26 13:09 Dockerfile
-rw-r--r-- 1 root root    405 Oct 26 11:18 Jenkinsfile
-rw-r--r-- 1 root root   3501 Oct 26 11:55 README.md
-rw-r--r-- 1 root root   7915 Oct 26 11:46 app.py
-rw-r--r-- 1 root root    799 Oct 26 11:19 jd.sh
-rw-r--r-- 1 root root   4224 Oct 26 10:22 note.txt
drwxr-xr-x 2 root root   4096 Oct 26 10:48 public
-rw-r--r-- 1 root root    146 Oct 26 11:17 requirements.txt
drwxr-xr-x 2 root root   4096 Oct 26 11:22 services
-rw-r--r-- 1 root root 305785 Oct 26 10:57 structure.txt
root@dc:~/srinu/pdf_edit# cat Dockerfile
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

# Explicitly copy ONLY the files and folders needed to run the app
COPY app.py ./
COPY services/ ./services/
COPY public/ ./public/

# Non-root user for security
RUN useradd -m appuser && chown -R appuser:appuser /usr/src/app
USER appuser

EXPOSE 3000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:3000", "app:app", "--timeout", "120"]



root@dc:~/srinu/pdf_edit# ll
total 372
drwxr-xr-x 5 root root   4096 Oct 26 18:39 ./
drwxr-xr-x 4 root root   4096 Oct 26 15:49 ../
-rw-r--r-- 1 root root    151 Oct 26 16:47 .dockerignore
-rw-r--r-- 1 root root    318 Oct 26 18:43 .env
-rw-r--r-- 1 root root    336 Oct 26 17:17 .env.gpg
drwxr-xr-x 7 root root   4096 Oct 26 14:57 .git/
-rw-r--r-- 1 root root    243 Oct 26 16:46 .gitignore
-rw-r--r-- 1 root root   1596 Oct 26 18:39 Dockerfile
-rw-r--r-- 1 root root    405 Oct 26 16:48 Jenkinsfile
-rw-r--r-- 1 root root   3501 Oct 26 17:25 README.md
-rw-r--r-- 1 root root   7915 Oct 26 17:16 app.py
-rw-r--r-- 1 root root    799 Oct 26 16:49 jd.sh
-rw-r--r-- 1 root root   4224 Oct 26 15:52 note.txt
drwxr-xr-x 2 root root   4096 Oct 26 16:18 public/
-rw-r--r-- 1 root root    146 Oct 26 16:47 requirements.txt
drwxr-xr-x 2 root root   4096 Oct 26 16:52 services/
-rw-r--r-- 1 root root 305785 Oct 26 16:27 structure.txt
root@dc:~/srinu/pdf_edit# 
