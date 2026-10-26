how to overcome this image build with nmo errors and warnings, not required ignorance, need solution?

Get:62 http://deb.debian.org/debian trixie/main amd64 sq amd64 1.3.1-2+b2 [5653 kB]
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC entries checked: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.40.1 /usr/local/share/perl/5.40.1 /usr/lib/x86_64-linux-gnu/perl5/5.40 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.40 /usr/share/perl/5.40 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 8, <STDIN> line 62.)
debconf: falling back to frontend: Teletype
debconf: unable to initialize frontend: Teletype
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Noninteractive
Fetched 92.0 MB in 1s (130 MB/s)
Selecting previously unselected package liblocale-gettext-perl.



Saved ./wheels/numpy-2.0.2-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
Saved ./wheels/requests-2.31.0-py3-none-any.whl
Saved ./wheels/gunicorn-21.2.0-py3-none-any.whl

[notice] A new release of pip is available: 24.0 -> 26.2.2
[notice] To update, run: pip install --upgrade pip
 ---> Removed intermediate container c88a570aef05
 ---> ab77dd977c6a
Step 6/16 : FROM python:3.11-slim

Get:104 http://deb.debian.org/debian trixie/main amd64 tesseract-ocr-tel all 1:4.1.0-2 [1015 kB]
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC entries checked: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.40.1 /usr/local/share/perl/5.40.1 /usr/lib/x86_64-linux-gnu/perl5/5.40 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.40 /usr/share/perl/5.40 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 8, <STDIN> line 104.)
debconf: falling back to frontend: Teletype
debconf: unable to initialize frontend: Teletype
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Noninteractive
Preconfiguring packages ...
Fetched 94.7 MB in 1s (121 MB/s)


Successfully installed Jinja2-3.1.6 MarkupSafe-3.0.4 PyMuPDF-1.24.9 PyMuPDFb-1.24.9 Werkzeug-3.1.9 blinker-1.9.0 certifi-2026.10.23 charset-normalizer-3.5.2 click-8.5.0 flask-3.0.3 gunicorn-21.2.0 idna-3.20 itsdangerous-2.2.0 numpy-2.0.2 opencv-python-headless-4.10.0.84 pillow-10.4.0 pytesseract-0.3.13 requests-2.31.0 urllib3-2.8.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 24.0 -> 26.2.2
[notice] To update, run: pip install --upgrade pip


root@dc:~/srinu/pdf_edit# docker exec -it app ls -la /usr/src/app
total 372
drwxr-xr-x 5 root root   4096 Oct 26 11:55 .
drwxr-xr-x 1 root root   4096 Oct 26 11:49 ..
-rw-r--r-- 1 root root    151 Oct 26 11:17 .dockerignore
-rw-r--r-- 1 root root    318 Oct 26 11:51 .env
-rw-r--r-- 1 root root    336 Oct 26 11:47 .env.gpg
drwxr-xr-x 7 root root   4096 Oct 26 09:27 .git
-rw-r--r-- 1 root root    243 Oct 26 11:16 .gitignore
-rw-r--r-- 1 root root   1176 Oct 26 11:18 Dockerfile
-rw-r--r-- 1 root root    405 Oct 26 11:18 Jenkinsfile
-rw-r--r-- 1 root root   3501 Oct 26 11:55 README.md
-rw-r--r-- 1 root root   7915 Oct 26 11:46 app.py
-rw-r--r-- 1 root root    799 Oct 26 11:19 jd.sh
-rw-r--r-- 1 root root   4224 Oct 26 10:22 note.txt
drwxr-xr-x 2 root root   4096 Oct 26 10:48 public
-rw-r--r-- 1 root root    146 Oct 26 11:17 requirements.txt
drwxr-xr-x 2 root root   4096 Oct 26 11:22 services
-rw-r--r-- 1 root root 305785 Oct 26 10:57 structure.txt
root@dc:~/srinu/pdf_edit# 
root@dc:~/srinu/pdf_edit# ll
total 372
drwxr-xr-x 5 root root   4096 Oct 26 17:25 ./
drwxr-xr-x 4 root root   4096 Oct 26 15:49 ../
-rw-r--r-- 1 root root    151 Oct 26 16:47 .dockerignore
-rw-r--r-- 1 root root    318 Oct 26 17:21 .env
-rw-r--r-- 1 root root    336 Oct 26 17:17 .env.gpg
drwxr-xr-x 7 root root   4096 Oct 26 14:57 .git/
-rw-r--r-- 1 root root    243 Oct 26 16:46 .gitignore
-rw-r--r-- 1 root root   1176 Oct 26 16:48 Dockerfile
-rw-r--r-- 1 root root    405 Oct 26 16:48 Jenkinsfile
-rw-r--r-- 1 root root   3501 Oct 26 17:25 README.md
-rw-r--r-- 1 root root   7915 Oct 26 17:16 app.py
-rw-r--r-- 1 root root    799 Oct 26 16:49 jd.sh
-rw-r--r-- 1 root root   4224 Oct 26 15:52 note.txt
drwxr-xr-x 2 root root   4096 Oct 26 16:18 public/
-rw-r--r-- 1 root root    146 Oct 26 16:47 requirements.txt
drwxr-xr-x 2 root root   4096 Oct 26 16:52 services/
-rw-r--r-- 1 root root 305785 Oct 26 16:27 structure.txt
root@dc:~/srinu/pdf_edit# cat .dockerignore 
.git
.gitignore
.env
.env.*
*.gpg
__pycache__/
*.pyc
*.pyo
.vscode/
.idea/
venv/
.venv/
tmp/
cache/
README.md
Jenkinsfile
jd.sh
structure.txt
note.txt
root@dc:~/srinu/pdf_edit# cat Dockerfile 
# ==========================================
# Stage 1: Builder (Heavy dependencies)
# ==========================================
FROM python:3.11-slim AS builder

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

RUN pip install --no-cache /wheels/*

COPY . .

# Non-root user for security
RUN useradd -m appuser && chown -R appuser:appuser /usr/src/app
USER appuser

EXPOSE 3000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:3000", "app:app", "--timeout", "120"]

root@dc:~/srinu/pdf_edit# 



i need light weight container, here all files are inside container some are not ignored, why?
