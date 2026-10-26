Here is the updated README.md reflecting your new modular architecture, UI enhancements, and the command cheat sheet extracted from your note.txt.
PixelEdit (Refactored & Optimized)

A modular Flask + OpenCV + Tesseract + PyMuPDF editor for precise text and image replacement.

#What's New in this Version

- **Modular Architecture:** Business logic is now decoupled into dedicated micro-services (`pdf_service.py`, `image_service.py`, `github_service.py`, and `session_service.py`).
- **Optimized UI/UX:** Features a new draggable custom navigation widget with dynamic transparency (30% resting, 60% active) for seamless viewport manipulation.
- **Lightweight Deployment:** Utilizes a multi-stage `Dockerfile` to drastically reduce image size, moving from a heavy builder stage to a clean runtime using Gunicorn.
- **Enhanced Security & CI/CD:** Hardened `.env` integration, strict `.gitignore`/`.dockerignore` rules, and an updated `Jenkinsfile` tracking the `main9` branch.

#Project Structure

```text
pdf_edit
├── app.py
├── .env
├── .gitignore
├── .dockerignore
├── requirements.txt
├── services
│   ├── pdf_service.py
│   ├── image_service.py
│   ├── github_service.py
│   └── session_service.py
├── public
│   ├── index.html
│   ├── script.js
│   ├── style.css
│   └── assets
├── Dockerfile
├── Jenkinsfile
└── jd.sh
```

Quick Start (Docker)
You can build and deploy the application using the included deployment script:
```bash
chmod +x jd.sh
./jd.sh
```

Or manually using Docker:
docker build -t srinuas/pdfrepo:latest .
docker run -d --name pdfapp --env-file .env -p 333:3000 srinuas/pdfrepo:latest

Then open http://localhost:333.
Developer Cheat Sheet
The following commands are frequently used for maintenance, security, and cleanup.
Environment Variable Security (GPG)
Encrypt .env file (Prompts for passphrase, e.g., 's')
```bash
gpg -c .env
```

Securely shred and delete the unencrypted .env file
```bash
shred -u .env
```

View contents without creating a file
```bash
gpg -d .env.gpg
```

Restore back to .env
```bash
gpg -d .env.gpg > .env
```
Docker Management & Cleanup
Stop and remove the active container
```bash
docker rm -f app
```

Remove specific images
```bash
docker rmi srinuas/pdfimg:1
docker rmi python:3.11-slim
```

Clean up EVERYTHING unused (Stopped containers, dangling images, etc.)
```bash
docker system prune -a
```

Create a persistent volume
```bash
docker volume create pixeledit_data
```

Automated Build, Decrypt, and Run:
```bash
export GP='s' && docker rm -f app && docker build --no-cache -t srinuas/pdfimg:1 . && gpg --batch --yes --passphrase "$GP" -d .env.gpg > .env && docker run -itd --name app -p 333:3000 --env-file .env -v /root/srinu/pdf_edit:/usr/src/app srinuas/pdfimg:1
```

Code Formatting
Format Python backend
```bash
black app.py services/
```

Format Frontend instantly using Node/Prettier
```bash
npx prettier --write "public/index.html" "public/style.css" "public/script.js"
```

Important Limitations
OCR-based editing is strongest on clear, horizontal text with reasonable contrast. For arbitrary decorative fonts, handwriting, curved text, complex backgrounds, or text overlapping graphics, pixel-perfect reconstruction cannot be guaranteed by OCR alone.
