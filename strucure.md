root@dc:~/srinu/pdf_edit# ll
total 44
drwxr-xr-x 4 root root 4096 Oct 26 20:17 ./
drwxr-xr-x 3 root root 4096 Oct 26 19:33 ../
-rw-r--r-- 1 root root  223 Oct 26 20:17 .env
-rw-r--r-- 1 root root  277 Oct 26 20:17 .env.gpg
drwxr-xr-x 7 root root 4096 Oct 26 20:09 .git/
-rw-r--r-- 1 root root  189 Oct 26 20:09 Dockerfile
-rw-r--r-- 1 root root 3839 Oct 26 20:09 app.py
-rw-r--r-- 1 root root  816 Oct 26 20:09 deploy_image.sh
-rw-r--r-- 1 root root  319 Oct 26 20:09 docker-compose.yml
drwxr-xr-x 2 root root 4096 Oct 26 20:22 public/
-rw-r--r-- 1 root root   29 Oct 26 20:09 requirements.txt
root@dc:~/srinu/pdf_edit# ll public/
total 844
drwxr-xr-x 2 root root   4096 Oct 26 20:22 ./
drwxr-xr-x 4 root root   4096 Oct 26 20:17 ../
-rw-r--r-- 1 root root 317942 Oct 26 20:14 bg_img_1.jpg
-rw-r--r-- 1 root root 306214 Oct 26 20:14 bg_img_2.jpg
-rw-r--r-- 1 root root  14704 Oct 26 20:14 bg_img_3.jpg
-rw-r--r-- 1 root root  98284 Oct 26 20:14 bg_img_4.jpg
-rw-r--r-- 1 root root  28793 Oct 26 20:14 favicon.jpg
-rw-r--r-- 1 root root  25396 Oct 26 20:14 index.html
-rw-r--r-- 1 root root  30252 Oct 26 20:14 script.js
-rw-r--r-- 1 root root  19543 Oct 26 20:14 style.css
root@dc:~/srinu/pdf_edit# tree .
.
├── Dockerfile
├── app.py
├── deploy_image.sh
├── docker-compose.yml
├── public
│   ├── bg_img_1.jpg
│   ├── bg_img_2.jpg
│   ├── bg_img_3.jpg
│   ├── bg_img_4.jpg
│   ├── favicon.jpg
│   ├── index.html
│   ├── script.js
│   └── style.css
└── requirements.txt

2 directories, 13 files
root@dc:~/srinu/pdf_edit# 
root@dc:~/srinu/pdf_edit# cat .env
GITHUB_TOKEN=xxx
GITHUB_REPO=Srinuas/photo-storage
GITHUB_BRANCH=main

ADMIN_PIN=1234
VAULT_PIN=4321
SECRET_KEY=my_super_secret_key

root@dc:~/srinu/pdf_edit# ll
total 44
root@dc:~/srinu/pdf_edit# tree .
.
├── Dockerfile
├── app.py
├── deploy_image.sh
├── docker-compose.yml
├── public
│   ├── bg.jpg
│   ├── bg_img_1.jpg
│   ├── bg_img_2.jpg
│   ├── bg_img_3.jpg
│   ├── bg_img_4.jpg
│   ├── favicon.jpg
│   ├── index.html
│   ├── script.js
│   └── style.css
└── requirements.txt

2 directories, 14 files
root@dc:~/srinu/pdf_edit# cat .env
GITHUB_TOKEN=xxxx
GITHUB_REPO=Srinuas/photo-storage
GITHUB_BRANCH=main

ADMIN_PIN=1234
VAULT_PIN=4321
SECRET_KEY=my_super_secret_key

root@dc:~/srinu/pdf_edit# 
root@dc:~/srinu/pdf_edit# sh /srinu/install/cat.sh 
Do you want to display line numbers? (y/n): n
========================================
FILE: ./app.py
========================================
import os
import fitz  # PyMuPDF
import json
from flask import Flask, request, send_file
import io

app = Flask(__name__, static_folder='public', static_url_path='')

@app.route('/')
def index():
    return app.send_static_file('index.html')

@app.route('/edit-pdf', methods=['POST'])
def edit_pdf():
    if 'pdfFile' not in request.files:
        return "No file uploaded", 400
        
    file = request.files['pdfFile']
    
    try:
        edits = json.loads(request.form.get('edits', '[]'))
    except json.JSONDecodeError:
        edits = []
        
    pdf_bytes = file.read()
    doc = fitz.open(stream=pdf_bytes, filetype="pdf")
    
    for page in doc:
        text_dict = page.get_text("dict")
        
        for block in text_dict.get("blocks", []):
            if block.get("type") == 0:
                for line in block.get("lines", []):
                    line_text = "".join([span["text"] for span in line.get("spans", [])])
                    
                    needs_edit = False
                    for edit in edits:
                        if edit.get('search') and edit['search'] in line_text:
                            needs_edit = True
                            break
                            
                    if needs_edit:
                        line_rect = fitz.Rect(line["bbox"])
                        page.draw_rect(line_rect, color=(1, 1, 1), fill=(1, 1, 1))
                        
                        current_x = line_rect.x0
                        
                        for span in line.get("spans", []):
                            span_text = span["text"]
                            
                            for edit in edits:
                                if edit.get('search') and edit['search'] in span_text:
                                    span_text = span_text.replace(edit['search'], edit.get('replace', ''))
                            
                            if not span_text:
                                continue
                                
                            font_size = span["size"]
                            flags = span["flags"]
                            color_int = span["color"]
                            
                            r = ((color_int >> 16) & 255) / 255.0
                            g = ((color_int >> 8) & 255) / 255.0
                            b = (color_int & 255) / 255.0
                            
                            is_bold = bool(flags & 16)
                            is_italic = bool(flags & 2)
                            
                            # CORRECTED: PyMuPDF Official Font Abbreviations
                            font_name = "helv"
                            if is_bold and is_italic:
                                font_name = "hebi"
                            elif is_bold:
                                font_name = "hebo"
                            elif is_italic:
                                font_name = "heit"
                            
                            page.insert_text(
                                fitz.Point(current_x, span["origin"][1]), 
                                span_text, 
                                fontsize=font_size, 
                                fontname=font_name, 
                                color=(r, g, b)
                            )
                            
                            text_length = fitz.get_text_length(span_text, fontname=font_name, fontsize=font_size)
                            current_x += text_length

    out_pdf = io.BytesIO()
    doc.save(out_pdf)
    doc.close()
    out_pdf.seek(0)
    
    return send_file(out_pdf, mimetype='application/pdf')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 3000))
    app.run(host='0.0.0.0', port=port)
========================================
FILE: ./requirements.txt
========================================
Flask==3.0.3
PyMuPDF==1.24.9

========================================
FILE: ./deploy_image.sh
========================================
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Configuration
DOCKER_USER="srinuas"
REPO_NAME="pdfrepo"
# Use the first argument as the tag, or default to "latest" if not provided
TAG=${1:-"latest"}
IMAGE_NAME="$DOCKER_USER/$REPO_NAME:$TAG"

echo "=== Building Docker image: $IMAGE_NAME ==="
docker build -t $IMAGE_NAME .

echo "=== Pushing image to Docker Hub ==="
docker push $IMAGE_NAME

echo "=== Success! $IMAGE_NAME is now on Docker Hub ==="

echo "=== Restarting Docker Swarm Stack ==="
docker stack rm pdf_stack

# Wait 15 seconds for the containers and network to fully terminate
sleep 15

docker stack deploy -c docker-compose.yml pdf_stack

# sh deploy_image.sh
# sh deploy_image.sh <version>
# docker service update --image srinuas/pdfrepo:latest --force pdf_stack_web
========================================
FILE: ./Dockerfile
========================================
FROM python:3.11-slim

WORKDIR /usr/src/app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 3000

# Run the Flask app
CMD ["python", "app.py"]

========================================
FILE: ./.env.gpg
========================================
?
?Ɵj?q~???Cթ??U¼Y?24/X??
                       ds?V[??޹|?Vmt@궇?0@U]
3gP?KkJ&8ߦN???|??w渰s?x?e????B	???3?????⧡q?jߏ?>1?;?I?Q??<F<?uwڶ?7
                                                                  ??)??TP{D:^??{??R?) ?1??k????(q?f???\?$?=$`?/u@Q???A??8KAKsD?瀸Xj????P)??.2???a?tt%??O??b??.?ڌ???W'???A3?
========================================
FILE: ./docker-compose.yml
========================================
version: '3.8'

services:
  web:
    image: srinuas/pdfrepo:latest
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
      - SECRET_KEY=your_local_secret_key
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

========================================
FILE: ./public/style.css
========================================
* {
  box-sizing: border-box;
}
html,
body {
  margin: 0;
  font-family: system-ui, sans-serif;
  background: #000;
  color: #eef2ff;
  overflow-x: hidden;
  width: 100%;
}

#appBackground {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-size: cover;
  background-position: center;
  transition: opacity 0.5s ease-in-out;
  filter: brightness(0.35) blur(6px);
  z-index: -1;
}

.app-shell {
  width: 100%;
  max-width: 1400px;
  margin: auto;
  padding: 20px;
  position: relative;
  z-index: 1;
}

.workspace {
  display: grid;
  grid-template-columns: minmax(300px, 380px) minmax(0, 850px);
  gap: 20px;
  align-items: start;
  width: 100%;
  justify-content: center;
}

.topbar {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.brand-mark {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #7c3aed, #06b6d4);
  display: grid;
  place-items: center;
  border-radius: 10px;
  font-weight: bold;
}
.panel {
  background: rgba(15, 20, 35, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 15px 50px rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  min-width: 0;
  width: 100%;
  overflow-x: hidden;
}

.slide-in {
  animation: slideIn 0.3s ease-out forwards;
}
.fade-in {
  animation: fadeIn 0.4s ease-out forwards;
}
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: none;
  }
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.toast {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background: #10b981;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  font-weight: bold;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
  z-index: 10000;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.nudge-btn {
  background: #1e293b;
  color: white;
  border: 1px solid #334155;
  border-radius: 8px;
  width: 40px;
  height: 40px;
  font-size: 16px;
  cursor: pointer;
  margin: 2px;
  transition: 0.2s;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
}
.nudge-btn:hover {
  background: #3b82f6;
  border-color: #60a5fa;
  transform: scale(1.05);
}
.nudge-btn:active {
  transform: scale(0.95);
}

.bold-btn {
  border-radius: 50%;
  background: #374151;
  font-weight: bold;
  font-size: 14px;
  width: 42px;
  height: 42px;
}

.color-swatch {
  width: 22px;
  height: 22px;
  border-radius: 4px;
  border: 1px solid #4b5563;
  cursor: pointer;
  transition: 0.2s;
  padding: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #000;
  font-weight: bold;
  text-shadow: 0 0 2px rgba(255, 255, 255, 0.8);
}
.color-swatch:hover {
  transform: scale(1.2);
  border-color: #fff;
}

.dropzone {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 150px;
  border: 2px dashed #35405e;
  border-radius: 12px;
  cursor: pointer;
  background: rgba(17, 23, 39, 0.7);
  position: relative;
}
.dropzone input {
  position: absolute;
  opacity: 0;
  inset: 0;
  cursor: pointer;
}

.mode-toggle,
.manual-tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.manual-tabs {
  margin-bottom: 10px;
}
.mode-btn,
.manual-tab {
  flex: 1;
  padding: 10px;
  background: #1a2235;
  border: 1px solid #2d3753;
  color: #8a96b1;
  border-radius: 8px;
  cursor: pointer;
  transition: 0.2s;
}
.mode-btn.active,
.manual-tab.active {
  background: #4f46e5;
  color: white;
  border-color: #4f46e5;
}

.edit-row {
  display: grid;
  grid-template-columns: 14px 1fr 20px;
  gap: 10px;
  margin-bottom: 10px;
  background: rgba(12, 18, 32, 0.8);
  padding: 10px;
  border-radius: 10px;
  border: 1px solid #202b45;
  cursor: grab;
  align-items: center;
}
.edit-row:active {
  cursor: grabbing;
}
.dragging-row {
  opacity: 0.4;
  border: 1px dashed #7c6cff;
  background: #1a2235;
}
.drag-handle {
  color: #4b5563;
  font-size: 16px;
  user-select: none;
}
.edit-fields {
  display: grid;
  grid-template-columns: 1fr 34px 1fr;
  gap: 6px;
  min-width: 0;
}
.edit-fields input {
  width: 100%;
  min-width: 0;
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #26314b;
  background: rgba(17, 24, 41, 0.8);
  color: white;
}

.swap-btn {
  background: rgba(124, 108, 255, 0.1);
  border: 1px solid #26314b;
  color: #7c6cff;
  border-radius: 8px;
  cursor: pointer;
  height: 34px;
  width: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: 0.2s;
}
.swap-btn:hover {
  background: #7c6cff;
  color: #fff;
  border-color: #7c6cff;
}
.remove-btn,
.text-btn {
  background: none;
  border: none;
  color: #ef4444;
  cursor: pointer;
  font-size: 20px;
}
.text-btn {
  color: #a78bfa;
  font-size: 14px;
}

.primary-btn,
.success-btn,
.secondary-btn,
.danger-btn,
.fatal-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.2s;
}
.primary-btn {
  background: linear-gradient(135deg, #6d5dfc, #4f46e5);
  color: white;
  margin-top: 15px;
}
.primary-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(79, 70, 229, 0.4);
}
.success-btn {
  background: #10b981;
  color: white;
}
.success-btn:hover {
  background: #059669;
}
.secondary-btn {
  background: #1f2937;
  color: #d1d5db;
  margin-top: 10px;
  border: 1px solid #374151;
}
.secondary-btn:hover {
  background: #374151;
  color: white;
}

.image-upload-btn {
  display: block;
  background: rgba(17, 24, 41, 0.8);
  border: 1px dashed #4f46e5;
  color: #a78bfa;
  text-align: center;
  padding: 15px;
  border-radius: 8px;
  cursor: pointer;
  transition: 0.2s;
}
.image-upload-btn:hover {
  background: rgba(79, 70, 229, 0.2);
}
.download-grid {
  display: flex;
  gap: 10px;
}

.preview-top {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 15px;
}
.history-controls,
.zoom-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

.tool-btn {
  background: #171e35;
  color: #a78bfa;
  border: 1px solid #26314b;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 12px;
  font-weight: bold;
  white-space: nowrap;
}
.tool-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}
.tool-btn:hover:not(:disabled) {
  background: #1f2a4a;
}
select.tool-btn {
  appearance: none;
  -webkit-appearance: none;
  background: #171e35
    url("data:image/svg+xml;utf8,<svg fill='%23a78bfa' height='24' viewBox='0 0 24 24' width='24' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/><path d='M0 0h24v24H0z' fill='none'/></svg>")
    no-repeat right 4px center;
  background-size: 18px;
  padding-right: 25px;
  cursor: pointer;
}

.canvas-viewport {
  width: 100%;
  max-width: 100%;
  max-height: 70vh;
  overflow: auto;
  background: rgba(15, 23, 42, 0.6);
  border-radius: 8px;
  border: 1px solid #202b45;
  position: relative;
  scroll-behavior: smooth;
  direction: ltr;
}
.canvas-viewport::-webkit-scrollbar {
  width: 12px;
  height: 12px;
}
.canvas-viewport::-webkit-scrollbar-track {
  background: rgba(15, 20, 35, 0.8);
  border-radius: 8px;
}
.canvas-viewport::-webkit-scrollbar-thumb {
  background: #4f46e5;
  border-radius: 8px;
  border: 2px solid rgba(15, 20, 35, 0.8);
}
.canvas-viewport::-webkit-scrollbar-thumb:hover {
  background: #6366f1;
}

.canvas-container {
  position: relative;
  display: inline-block;
  transform-origin: top left;
  transition: transform 0.2s ease-out;
}
#previewImg {
  display: block;
  max-width: none;
  border-radius: 8px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
}
#drawCanvas {
  position: absolute;
  top: 0;
  left: 0;
  cursor: crosshair;
}

.hidden {
  display: none !important;
}
.info-box {
  font-size: 12px;
  color: #94a3b8;
  background: #131a2c;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 15px;
  line-height: 1.5;
}
.manual-form input[type="text"] {
  width: 100%;
  padding: 10px;
  margin-top: 5px;
  border-radius: 6px;
  border: 1px solid #26314b;
  background: rgba(17, 24, 41, 0.8);
  color: white;
}
.divider {
  border-color: #202b45;
  margin: 25px 0;
}
input[type="color"]::-webkit-color-swatch-wrapper {
  padding: 0;
  border-radius: 6px;
}
input[type="color"]::-webkit-color-swatch {
  border: none;
  border-radius: 6px;
}

.developer-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
  padding: 15px 25px;
  background: rgba(15, 20, 35, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  color: #94a3b8;
  font-size: 13px;
  font-weight: 500;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
}
.contact-btn {
  background: #4f46e5;
  color: white;
  text-decoration: none;
  padding: 6px 14px;
  border-radius: 6px;
  font-weight: bold;
  font-size: 12px;
  transition: 0.2s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
}
.contact-btn:hover {
  background: #6366f1;
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(99, 102, 241, 0.4);
}

.admin-trigger {
  position: fixed;
  bottom: 20px;
  right: 20px;
  font-size: 28px;
  opacity: 0.2;
  cursor: pointer;
  z-index: 1000;
  transition: 0.3s;
}
.admin-trigger:hover {
  opacity: 1;
  transform: scale(1.1);
}

.admin-modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(8px);
  z-index: 9999;
  display: grid;
  place-items: center;
  padding: 20px;
}
.admin-content,
.preview-modal-content {
  background: rgba(11, 17, 32, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  width: 100%;
  max-width: 1000px;
  height: 80vh;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(12px);
}
.preview-modal-content {
  max-width: 1200px;
  height: 90vh;
}
.admin-header,
.preview-header {
  padding: 20px;
  border-bottom: 1px solid #202b45;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(15, 23, 42, 0.8);
}
.admin-header h2 {
  margin: 0;
  font-size: 18px;
  background: linear-gradient(135deg, #a78bfa, #22d3ee);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
.preview-header h3 {
  margin: 0;
  font-size: 16px;
  color: #e2e8f0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 90%;
}
.close-admin {
  background: none;
  border: none;
  color: #9ca3af;
  font-size: 24px;
  cursor: pointer;
}
.close-admin:hover {
  color: white;
}

.admin-layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  flex: 1;
  overflow: hidden;
}
.admin-sidebar {
  background: rgba(19, 26, 44, 0.8);
  border-right: 1px solid #202b45;
  padding: 15px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  overflow-y: auto;
}
.admin-tab {
  background: none;
  border: none;
  color: #9ca3af;
  text-align: left;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: 0.2s;
  font-size: 13px;
}
.admin-tab:hover {
  background: #1e293b;
  color: white;
}
.admin-tab.active {
  background: #2563eb;
  color: white;
}
.trash-tab.active {
  background: #dc2626;
}
.admin-main {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.admin-toolbar {
  padding: 15px;
  border-bottom: 1px solid #202b45;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
  justify-content: space-between;
  background: rgba(15, 23, 42, 0.8);
}
.admin-toolbar > * {
  flex: 1 1 auto;
}

.admin-grid {
  flex: 1;
  padding: 20px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 15px;
  overflow-y: auto;
}
.admin-file-card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 12px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  position: relative;
  cursor: pointer;
  transition: 0.2s;
}
.admin-file-card:hover {
  border-color: #60a5fa;
  transform: translateY(-2px);
}
.admin-chk {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 10;
  width: 16px;
  height: 16px;
  cursor: pointer;
}
.file-visual {
  height: 100px;
  background: rgba(15, 23, 42, 0.6);
  border-radius: 8px;
  display: grid;
  place-items: center;
  overflow: hidden;
  position: relative;
}
.file-visual img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}
.pdf-icon {
  font-size: 24px;
  font-weight: bold;
  color: #ef4444;
}
.file-name {
  font-size: 11px;
  color: #e2e8f0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: center;
}

.file-actions {
  position: absolute;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  opacity: 0;
  transition: all 0.2s ease;
  background: rgba(15, 23, 42, 0.85);
  z-index: 20;
}
.file-visual:hover .file-actions {
  opacity: 1;
}
.action-hover-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #0f172a;
  color: #60a5fa;
  border: 1px solid #60a5fa;
  font-size: 16px;
  cursor: pointer;
  display: grid;
  place-items: center;
  transition: all 0.2s ease;
  transform: scale(0.9);
}
.action-hover-btn:hover {
  background: #60a5fa;
  color: #fff;
  transform: scale(1.1);
}

.preview-body {
  flex: 1;
  background: transparent;
  display: grid;
  place-items: center;
  padding: 20px;
  position: relative;
}
.danger-btn {
  background: #b91c1c;
  color: white;
  width: auto;
  padding: 8px 16px;
  font-size: 12px;
  margin: 0;
}
.danger-btn:hover {
  background: #991b1b;
}
.fatal-btn {
  background: #000;
  color: #ef4444;
  border: 1px solid #ef4444;
  width: auto;
  padding: 8px 16px;
  font-size: 12px;
  margin: 0;
}
.fatal-btn:hover {
  background: #ef4444;
  color: white;
}

.loader {
  position: absolute;
  inset: 0;
  background: rgba(15, 23, 42, 0.7);
  backdrop-filter: blur(2px);
  z-index: 20;
}
.loader::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  margin: -20px 0 0 -20px;
  width: 40px;
  height: 40px;
  border: 4px solid #334155;
  border-top-color: #60a5fa;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
.scale-in {
  animation: scaleIn 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-8px);
  }
  50% {
    transform: translateX(8px);
  }
  75% {
    transform: translateX(-8px);
  }
}
.shake {
  animation: shake 0.4s ease-in-out;
}
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.pin-content {
  background: rgba(15, 23, 42, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 30px;
  border-radius: 20px;
  width: 100%;
  max-width: 360px;
  text-align: center;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(12px);
}
.pin-content h3 {
  font-size: 22px;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #a78bfa, #22d3ee);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
.pin-content p {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 24px;
}
#pinInput {
  width: 100%;
  padding: 15px;
  font-size: 28px;
  text-align: center;
  letter-spacing: 12px;
  border-radius: 12px;
  border: 1px solid #26314b;
  background: rgba(12, 18, 32, 0.8);
  color: white;
  margin-bottom: 24px;
  transition: 0.3s;
  outline: none;
}
#pinInput:focus {
  border-color: #6366f1;
  box-shadow: 0 0 0 4px rgba(99, 102, 241, 0.2);
}
.pin-actions {
  display: flex;
  gap: 12px;
}
.pin-actions button {
  margin-top: 0;
}
.secret-trigger {
  text-align: center;
  font-size: 16px;
  opacity: 0.15;
  cursor: pointer;
  padding: 10px;
  margin-top: auto;
  transition: all 0.3s ease;
  filter: grayscale(100%);
}
.secret-trigger:hover {
  opacity: 1;
  filter: grayscale(0%);
  transform: scale(1.2);
}
.empty-admin {
  grid-column: 1 / -1;
  text-align: center;
  padding: 40px;
  color: #64748b;
}

#navigationWidget {
  position: fixed;
  left: 20px;
  bottom: 20px;
  z-index: 9999;
  background: rgba(15, 23, 42, 0.45);
  backdrop-filter: blur(8px);
  border-radius: 12px;
  padding: 12px;
  opacity: 0.45;
  transition: 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  gap: 10px;
}
#navigationWidget:hover {
  opacity: 0.95;
  background: rgba(15, 23, 42, 0.9);
}
.nav-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}
.nav-title {
  font-size: 10px;
  font-weight: 700;
  color: #a78bfa;
  letter-spacing: 1px;
}
.nav-row {
  display: flex;
  gap: 5px;
  justify-content: center;
}
.nav-btn {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  cursor: pointer;
  font-size: 14px;
  display: grid;
  place-items: center;
  transition: 0.2s;
}
.nav-btn:hover {
  background: #4f46e5;
  transform: scale(1.1);
}
.widget-divider {
  width: 100%;
  height: 1px;
  background: rgba(255, 255, 255, 0.1);
  margin: 2px 0;
}
#pageCounter {
  font-size: 12px;
  font-weight: bold;
  color: white;
  min-width: 45px;
  text-align: center;
}

@media (max-width: 900px) {
  .workspace {
    display: flex;
    flex-direction: column;
    width: 100%;
  }
  .admin-layout {
    display: flex;
    flex-direction: column;
    height: auto;
  }

  /* CRITICAL MOBILE FIX: Scrollable Tabs */
  .admin-sidebar {
    flex-direction: row;
    overflow-x: auto;
    padding: 12px;
    border-right: none;
    border-bottom: 1px solid #202b45;
    -webkit-overflow-scrolling: touch;
    scrollbar-width: none; /* Hide scrollbar Firefox */
  }
  .admin-sidebar::-webkit-scrollbar {
    display: none;
  } /* Hide scrollbar WebKit */
  .admin-tab {
    white-space: nowrap;
    flex-shrink: 0; /* Prevents text from being squished */
    padding: 10px 16px;
  }
  .secret-trigger {
    margin-top: 0;
    margin-left: auto;
    display: flex;
    align-items: center;
    flex-shrink: 0;
  }
}

@media (max-width: 600px) {
  .app-shell {
    padding: 10px;
  }
  .panel {
    padding: 15px;
  }
  .edit-fields {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  .swap-btn {
    transform: rotate(90deg);
    margin: 0 auto;
  }
  .admin-content,
  .preview-modal-content {
    height: 95vh;
    border-radius: 12px;
  }

  /* CLEAN MOBILE DASHBOARD BUTTON GRID */
  .admin-toolbar {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    align-items: stretch;
  }
  #sortSelect,
  #adminPermDeleteBtn {
    grid-column: 1 / -1;
    width: 100%;
  }
  .admin-toolbar button,
  .admin-toolbar select {
    flex: none;
    width: 100%;
    margin: 0;
    padding: 10px;
  }

  .pin-content {
    padding: 20px;
  }
  .download-grid {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .preview-top {
    flex-direction: column;
    align-items: stretch;
    gap: 10px;
  }
  .history-controls {
    flex-wrap: wrap;
    justify-content: flex-start;
    margin-top: 5px;
  }
  .canvas-viewport {
    max-height: 55vh;
  }
  #navigationWidget {
    bottom: 10px;
    left: 10px;
    padding: 8px;
  }
}

========================================
FILE: ./public/index.html
========================================
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    <title>PixelEdit — Interactive Canvas Engine - Srinu Invention</title>
    <link rel="icon" type="image/jpeg" href="favicon.jpg" />
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div id="appBackground"></div>
    <div class="ambient ambient-a"></div>
    <div class="ambient ambient-b"></div>

    <div id="toastNotification" class="toast hidden" style="opacity: 0">
      Changes Applied Successfully!
    </div>

    <div class="app-shell">
      <header class="topbar">
        <div class="brand">
          <div class="brand-mark">✦</div>
          <div>
            <h1>PixelEdit - Srinu Invention</h1>
            <p>Stateful Memory & Manual Region Engine</p>
          </div>
        </div>
      </header>

      <main class="workspace">
        <!-- LEFT SIDEBAR -->
        <section class="panel controls">
          <div id="uploadSection">
            <div class="section-title"><h2>01. Upload Document</h2></div>
            <label class="dropzone" id="dropzone">
              <input id="fileInput" type="file" accept=".pdf,image/*" />
              <div class="upload-icon">↑</div>
              <strong>Drop PDF or image here</strong>
            </label>
          </div>

          <div id="editSection" class="hidden slide-in">
            <div class="mode-toggle">
              <button class="mode-btn active" id="btnAutoMode">
                Find & Replace
              </button>
              <button class="mode-btn" id="btnManualMode">
                Manual Selection
              </button>
            </div>

            <div id="autoUI">
              <div class="section-title edits-head">
                <h2>Auto Replace</h2>
                <button class="text-btn" id="addBtn">+ Add</button>
              </div>
              <div id="edits"></div>
              <button
                class="primary-btn apply-btn"
                onclick="submitEdit('auto', false)"
              >
                Replace & Save Temporarily
              </button>
            </div>

            <div id="manualUI" class="hidden">
              <div class="section-title edits-head"><h2>Region Editor</h2></div>
              <div class="tip">
                Draw a box on the document preview to extract and replace text
                or images.
              </div>

              <div id="manualForm" class="hidden manual-form">
                <div class="info-box" id="scanResults">Scanned: 0 Lines</div>

                <div class="manual-tabs">
                  <button
                    id="tabText"
                    class="manual-tab active"
                    onclick="setManualTab('text')"
                  >
                    Text
                  </button>
                  <button
                    id="tabImage"
                    class="manual-tab"
                    onclick="setManualTab('image')"
                  >
                    Image
                  </button>
                </div>

                <div id="manualTextInput">
                  <label
                    style="
                      font-size: 11px;
                      color: #a78bfa;
                      font-weight: bold;
                      margin-bottom: 2px;
                      display: block;
                      text-align: left;
                    "
                    >Target Text inside box (Optional for precision):</label
                  >
                  <input
                    type="text"
                    id="manualSearchInput"
                    placeholder="e.g. NT"
                    style="margin-bottom: 10px"
                  />

                  <div
                    style="
                      display: flex;
                      gap: 10px;
                      align-items: flex-start;
                      margin-bottom: 5px;
                    "
                  >
                    <div style="flex: 1">
                      <label
                        style="
                          font-size: 11px;
                          color: #a78bfa;
                          font-weight: bold;
                          margin-bottom: 2px;
                          display: block;
                          text-align: left;
                        "
                        >Replace With:</label
                      >
                      <input
                        type="text"
                        id="manualReplaceInput"
                        placeholder="e.g. GT"
                        style="margin-top: 0"
                      />
                    </div>
                    <div>
                      <label
                        style="
                          font-size: 11px;
                          color: #a78bfa;
                          font-weight: bold;
                          margin-bottom: 2px;
                          display: block;
                          text-align: center;
                        "
                        >Color:</label
                      >
                      <input
                        type="color"
                        id="manualTextColor"
                        style="
                          height: 38px;
                          width: 45px;
                          padding: 0;
                          border: 1px solid #26314b;
                          border-radius: 6px;
                          cursor: pointer;
                          background: #111829;
                        "
                        title="Override Text Color"
                      />
                    </div>
                  </div>

                  <!-- QUICK COLOR SWATCHES -->
                  <div style="display: flex; gap: 5px; margin-bottom: 10px">
                    <button
                      class="color-swatch"
                      style="background: #000000"
                      onclick="setColor('#000000')"
                    ></button>
                    <button
                      class="color-swatch"
                      style="background: #ef4444"
                      onclick="setColor('#ef4444')"
                    ></button>
                    <button
                      class="color-swatch"
                      style="background: #3b82f6"
                      onclick="setColor('#3b82f6')"
                    ></button>
                    <button
                      class="color-swatch"
                      style="background: #10b981"
                      onclick="setColor('#10b981')"
                    ></button>
                    <button
                      class="color-swatch"
                      id="originalColorSwatch"
                      style="background: #ffffff"
                      title="Reset to Original Extracted Color"
                      onclick="setColor(this.dataset.color || '#000000')"
                    >
                      ↻
                    </button>
                  </div>

                  <input type="hidden" id="manualOffsetX" value="0" />
                  <input type="hidden" id="manualOffsetY" value="0" />
                  <input type="hidden" id="manualOffsetSize" value="0" />
                  <input type="hidden" id="manualOffsetBold" value="0" />
                </div>

                <div id="manualImageInput" class="hidden">
                  <label class="image-upload-btn" for="manualReplaceImage">
                    <span id="imageSelectName">Choose Image...</span>
                    <input
                      type="file"
                      id="manualReplaceImage"
                      accept="image/*"
                      class="hidden"
                    />
                  </label>
                </div>

                <button
                  id="previewEditBtn"
                  class="primary-btn apply-btn"
                  onclick="submitEdit('manual', true)"
                >
                  1. Preview Replace
                </button>

                <!-- D-PAD & BOLD STRENGTH ZOOM -->
                <div
                  id="nudgeBlock"
                  class="hidden"
                  style="
                    margin-top: 15px;
                    background: #131a2c;
                    padding: 15px;
                    border-radius: 8px;
                    border: 1px solid #202b45;
                    text-align: center;
                  "
                >
                  <label
                    style="
                      font-size: 11px;
                      color: #a78bfa;
                      font-weight: bold;
                      margin-bottom: 10px;
                      display: block;
                    "
                    >Adjust Exact Position (0.5px Micro-Steps):</label
                  >

                  <div
                    style="
                      display: flex;
                      justify-content: center;
                      align-items: center;
                      gap: 20px;
                      margin-bottom: 15px;
                    "
                  >
                    <div
                      style="
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        gap: 5px;
                      "
                    >
                      <button
                        class="nudge-btn bold-btn"
                        onclick="nudgeBold(-1)"
                        title="Decrease Boldness (Up to -4)"
                      >
                        B-
                      </button>
                    </div>

                    <div
                      class="d-pad"
                      style="
                        display: grid;
                        grid-template-areas: &quot;. up .&quot; &quot;left . right&quot; &quot;. down .&quot;;
                        gap: 4px;
                      "
                    >
                      <button
                        class="nudge-btn"
                        style="grid-area: up"
                        onclick="nudgePos(0, -0.5)"
                        title="Move Up"
                      >
                        ▲
                      </button>
                      <button
                        class="nudge-btn"
                        style="grid-area: left"
                        onclick="nudgePos(-0.5, 0)"
                        title="Move Left"
                      >
                        ◀
                      </button>
                      <button
                        class="nudge-btn"
                        style="grid-area: right"
                        onclick="nudgePos(0.5, 0)"
                        title="Move Right"
                      >
                        ▶
                      </button>
                      <button
                        class="nudge-btn"
                        style="grid-area: down"
                        onclick="nudgePos(0, 0.5)"
                        title="Move Down"
                      >
                        ▼
                      </button>
                    </div>

                    <div
                      style="
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        gap: 5px;
                      "
                    >
                      <button
                        class="nudge-btn bold-btn"
                        onclick="nudgeBold(1)"
                        title="Increase Boldness (Up to +4)"
                      >
                        B+
                      </button>
                    </div>
                  </div>

                  <div
                    id="boldLevelDisplay"
                    style="
                      color: #a78bfa;
                      font-size: 11px;
                      font-weight: bold;
                      margin-bottom: 10px;
                    "
                  >
                    Weight: Normal
                  </div>

                  <div
                    style="
                      display: flex;
                      justify-content: center;
                      gap: 10px;
                      margin-bottom: 10px;
                    "
                  >
                    <button
                      class="tool-btn"
                      onclick="nudgeSize(0.5)"
                      title="Scale Up"
                      style="flex: 1; padding: 8px"
                    >
                      🔍 Zoom In (+)
                    </button>
                    <button
                      class="tool-btn"
                      onclick="nudgeSize(-0.5)"
                      title="Scale Down"
                      style="flex: 1; padding: 8px"
                    >
                      🔍 Zoom Out (-)
                    </button>
                  </div>

                  <button
                    class="success-btn apply-btn"
                    style="margin-top: 10px"
                    onclick="submitEdit('manual', false)"
                  >
                    2. Confirm & Save Edit
                  </button>
                  <button
                    class="secondary-btn apply-btn"
                    style="margin-top: 5px"
                    onclick="cancelPreview()"
                  >
                    Cancel Edit
                  </button>
                </div>
              </div>
            </div>

            <hr class="divider" />
            <div class="download-grid">
              <button
                class="success-btn"
                id="downloadBtn"
                onclick="downloadFile()"
              >
                Download Final File ↓
              </button>
              <button
                class="secondary-btn"
                id="newEditBtn"
                onclick="clearFile()"
              >
                Upload New File ⟲
              </button>
            </div>
          </div>
        </section>

        <!-- RIGHT PANEL: CANVAS VIEWPORT -->
        <section class="panel preview">
          <div class="preview-top">
            <h2>Interactive Canvas - Srinu Invention</h2>
            <div class="history-controls">
              <div class="zoom-controls">
                <button
                  class="tool-btn"
                  onclick="setZoom(0.2)"
                  title="Zoom In Canvas"
                >
                  🔍 +
                </button>
                <button
                  class="tool-btn"
                  onclick="setZoom(-0.2)"
                  title="Zoom Out Canvas"
                >
                  🔍 -
                </button>
                <button
                  class="tool-btn"
                  onclick="resetZoom()"
                  title="Reset Zoom"
                >
                  🔄
                </button>
              </div>
              <button
                class="tool-btn"
                id="undoBtn"
                disabled
                title="Undo Action"
              >
                ↶ Undo
              </button>
              <button
                class="tool-btn"
                id="redoBtn"
                disabled
                title="Redo Action"
              >
                ↷ Redo
              </button>
            </div>
          </div>

          <div class="viewer" id="viewer">
            <div class="empty-state" id="emptyState">
              Upload a file to begin stateful editing.
            </div>

            <div class="canvas-viewport hidden fade-in" id="canvasViewport">
              <div class="canvas-container" id="canvasContainer">
                <img id="previewImg" draggable="false" />
                <canvas id="drawCanvas"></canvas>
              </div>
            </div>
          </div>
        </section>
      </main>

      <!-- DEVELOPER SIGNATURE FOOTER -->
      <footer class="developer-footer fade-in">
        <span>This web page developed by Srinu V</span>
        <a
          href="https://portfolio-rbu2.onrender.com/"
          target="_blank"
          class="contact-btn"
          >Contact Here</a
        >
      </footer>
    </div>

    <!-- COMBINED NAVIGATION WIDGET -->
    <div id="navigationWidget">
      <div class="nav-section">
        <div class="nav-title">WEBPAGE</div>
        <div class="nav-row">
          <button class="nav-btn" onclick="scrollWindow('up')">▲</button>
        </div>
        <div class="nav-row">
          <button class="nav-btn" onclick="scrollWindow('left')">◀</button>
          <button class="nav-btn" onclick="scrollWindow('down')">▼</button>
          <button class="nav-btn" onclick="scrollWindow('right')">▶</button>
        </div>
      </div>

      <div class="widget-divider"></div>

      <div class="nav-section">
        <div class="nav-title">CANVAS</div>
        <div class="nav-row">
          <button class="nav-btn" onclick="scrollCanvas('up')">▲</button>
        </div>
        <div class="nav-row">
          <button class="nav-btn" onclick="scrollCanvas('left')">◀</button>
          <button class="nav-btn" onclick="scrollCanvas('down')">▼</button>
          <button class="nav-btn" onclick="scrollCanvas('right')">▶</button>
        </div>
      </div>

      <div class="widget-divider"></div>

      <div class="nav-section">
        <div class="nav-title">PDF PAGE</div>
        <div class="nav-row" style="align-items: center; gap: 8px">
          <button class="nav-btn" onclick="changePage('prev')">◀</button>
          <span id="pageCounter">1 / 1</span>
          <button class="nav-btn" onclick="changePage('next')">▶</button>
        </div>
      </div>
    </div>

    <div id="adminBtn" class="admin-trigger" onclick="openAdmin()">🛡️</div>

    <!-- MODALS -->
    <div id="pinModal" class="admin-modal hidden">
      <div class="pin-content scale-in">
        <h3>Admin Access</h3>
        <p>Enter your PIN to access the Dashboard</p>
        <input
          type="password"
          id="pinInput"
          placeholder="••••"
          maxlength="4"
          autocomplete="off"
        />
        <div class="pin-actions">
          <button class="secondary-btn" onclick="closePinModal()">
            Cancel
          </button>
          <button class="primary-btn" id="submitPinBtn" onclick="submitPin()">
            Unlock 🔓
          </button>
        </div>
      </div>
    </div>

    <div id="vaultPinModal" class="admin-modal hidden" style="z-index: 10005">
      <div class="pin-content scale-in">
        <h3>Vault Access</h3>
        <p>Enter Vault PIN to access Deleted files</p>
        <input
          type="password"
          id="vaultPinInput"
          placeholder="••••"
          maxlength="4"
          autocomplete="off"
        />
        <div class="pin-actions">
          <button class="secondary-btn" onclick="closeVaultPinModal()">
            Cancel
          </button>
          <button
            class="primary-btn"
            id="submitVaultPinBtn"
            onclick="submitVaultPin()"
          >
            Unlock 🔓
          </button>
        </div>
      </div>
    </div>

    <div id="adminModal" class="admin-modal hidden" style="z-index: 10000">
      <div class="admin-content fade-in">
        <div class="admin-header">
          <h2>GitHub Storage Dashboard</h2>
          <button class="close-admin" onclick="closeAdmin()">×</button>
        </div>

        <div class="admin-layout">
          <div class="admin-sidebar">
            <button
              class="admin-tab active"
              onclick="loadAdminTab('Uploaded', 'Images')"
            >
              Uploaded Images
            </button>
            <button
              class="admin-tab"
              onclick="loadAdminTab('Uploaded', 'PDFs')"
            >
              Uploaded PDFs
            </button>
            <button
              class="admin-tab"
              onclick="loadAdminTab('Downloaded', 'Images')"
            >
              Downloaded Images
            </button>
            <button
              class="admin-tab"
              onclick="loadAdminTab('Downloaded', 'PDFs')"
            >
              Downloaded PDFs
            </button>
            <button
              class="admin-tab trash-tab hidden"
              onclick="loadAdminTab('Deleted', 'Images')"
            >
              Deleted Images
            </button>
            <button
              class="admin-tab trash-tab hidden"
              onclick="loadAdminTab('Deleted', 'PDFs')"
            >
              Deleted PDFs
            </button>
            <div style="flex: 1"></div>
            <div
              class="secret-trigger"
              onclick="toggleTrashView()"
              title="Vault"
            >
              🐦‍🔥
            </div>
          </div>

          <div class="admin-main">
            <div class="admin-toolbar">
              <button class="tool-btn" onclick="selectAll(true)">
                Select All
              </button>
              <button class="tool-btn" onclick="selectAll(false)">
                Unselect All
              </button>
              <select
                id="sortSelect"
                class="tool-btn"
                onchange="renderAdminGrid()"
              >
                <option value="date-desc">Time: Newest First</option>
                <option value="date-asc">Time: Oldest First</option>
                <option value="name-asc">Name: A-Z</option>
                <option value="name-desc">Name: Z-A</option>
              </select>
              <button
                class="success-btn"
                id="adminDownloadBtn"
                onclick="downloadSelected()"
                style="
                  margin: 0;
                  padding: 8px 16px;
                  font-size: 12px;
                  width: auto;
                "
              >
                ⬇️ Download Selected
              </button>
              <button
                class="primary-btn hidden"
                id="adminRestoreBtn"
                onclick="executeAdminAction('restore')"
                style="
                  margin: 0;
                  padding: 8px 16px;
                  font-size: 12px;
                  width: auto;
                "
              >
                ♻️ Restore Selected
              </button>
              <button
                class="danger-btn"
                id="adminTrashBtn"
                onclick="executeAdminAction('trash')"
              >
                Delete Selected
              </button>
              <button
                class="fatal-btn hidden"
                id="adminPermDeleteBtn"
                onclick="executeAdminAction('delete')"
              >
                🐦‍🔥 Permanently Delete
              </button>
            </div>
            <div class="admin-grid" id="adminGrid"></div>
            <div id="adminLoader" class="loader hidden"></div>
          </div>
        </div>
      </div>
    </div>

    <div id="confirmModal" class="admin-modal hidden" style="z-index: 10005">
      <div class="pin-content scale-in">
        <div style="font-size: 48px; margin-bottom: 15px">⚠️</div>
        <h3
          style="
            background: none;
            -webkit-text-fill-color: #ef4444;
            color: #ef4444;
          "
        >
          Are you sure?
        </h3>
        <p>Permanently delete these files? This cannot be undone.</p>
        <div class="pin-actions">
          <button class="secondary-btn" id="confirmCancelBtn">Cancel</button>
          <button class="fatal-btn" id="confirmYesBtn">Delete 🐦‍🔥</button>
        </div>
      </div>
    </div>

    <div id="alertModal" class="admin-modal hidden" style="z-index: 10006">
      <div class="pin-content scale-in">
        <div style="font-size: 48px; margin-bottom: 15px">ℹ️</div>
        <h3
          style="
            background: none;
            color: #60a5fa;
            -webkit-text-fill-color: #60a5fa;
          "
          id="alertTitle"
        >
          Notice
        </h3>
        <p id="alertMessage">Message goes here.</p>
        <div class="pin-actions" style="justify-content: center">
          <button
            class="primary-btn"
            onclick="closeCustomAlert()"
            style="width: 50%"
          >
            OK
          </button>
        </div>
      </div>
    </div>

    <div
      id="filePreviewModal"
      class="admin-modal hidden"
      style="z-index: 10001"
    >
      <div class="preview-modal-content scale-in">
        <div class="preview-header">
          <h3 id="previewTitle">Document Preview</h3>
          <button class="close-admin" onclick="closeFilePreview()">×</button>
        </div>
        <div class="preview-body" id="previewContainer"></div>
      </div>
    </div>

    <script src="script.js"></script>
  </body>
</html>

========================================
FILE: ./public/script.js
========================================
const $ = (s) => document.querySelector(s);
const $$ = (s) => document.querySelectorAll(s);

let session = { id: null, nativeWidth: 0, nativeHeight: 0 };
let currentMode = "auto";
let manualType = "text";
let manualBoxNative = null;
let suggestedFontSize = 12;
let manualTextProps = { angle: 0, is_bold: false, color: [0, 0, 0] };
let manualReplaceImageBase64 = null;
let currentZoom = 1;
let currentPage = 1;
let totalPages = 1;
let currentAdminFiles = []; // Holds fetched files for sorting

const fileInput = $("#fileInput");
const uploadSection = $("#uploadSection");
const editSection = $("#editSection");
const previewImg = $("#previewImg");
const drawCanvas = $("#drawCanvas");
const canvasContainer = $("#canvasContainer");
const canvasViewport = $("#canvasViewport");
const emptyState = $("#emptyState");
const undoBtn = $("#undoBtn");
const redoBtn = $("#redoBtn");
const newEditBtn = $("#newEditBtn");

let currentBg = "bg_img_1.jpg";

function setBackground(imgName) {
  if (currentBg === imgName && $("#appBackground").style.backgroundImage)
    return;
  currentBg = imgName;
  const bgDiv = document.getElementById("appBackground");
  bgDiv.style.opacity = 0;
  setTimeout(() => {
    bgDiv.style.backgroundImage = `url('${imgName}')`;
    bgDiv.style.opacity = 1;
  }, 500);
}

document.addEventListener("DOMContentLoaded", () => {
  setBackground("bg_img_1.jpg");
});

window.showCustomAlert = function (message, title = "Notice") {
  $("#alertTitle").textContent = title;
  $("#alertMessage").textContent = message;
  $("#alertModal").classList.remove("hidden");
};

window.closeCustomAlert = function () {
  $("#alertModal").classList.add("hidden");
};

window.showToast = function (msg) {
  const toast = $("#toastNotification");
  toast.textContent = msg;
  toast.classList.remove("hidden");
  toast.style.opacity = 1;
  setTimeout(() => {
    toast.style.opacity = 0;
    setTimeout(() => toast.classList.add("hidden"), 300);
  }, 6000);
};

window.setZoom = function (step) {
  currentZoom = Math.max(0.2, Math.min(4.0, currentZoom + step));
  canvasContainer.style.transform = `scale(${currentZoom})`;
};

window.resetZoom = function () {
  currentZoom = 1;
  canvasContainer.style.transform = `scale(1)`;
};

window.scrollWindow = function (dir) {
  const step = 300;
  if (dir === "up") window.scrollBy({ top: -step, behavior: "smooth" });
  if (dir === "down") window.scrollBy({ top: step, behavior: "smooth" });
  if (dir === "left") window.scrollBy({ left: -step, behavior: "smooth" });
  if (dir === "right") window.scrollBy({ left: step, behavior: "smooth" });
};

window.scrollCanvas = function (dir) {
  const vp = document.getElementById("canvasViewport");
  const step = 200;
  if (dir === "up") vp.scrollBy({ top: -step, behavior: "smooth" });
  if (dir === "down") vp.scrollBy({ top: step, behavior: "smooth" });
  if (dir === "left") vp.scrollBy({ left: -step, behavior: "smooth" });
  if (dir === "right") vp.scrollBy({ left: step, behavior: "smooth" });
};

window.changePage = async function (action) {
  if (!session.id) return;
  const res = await fetch("/api/page", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ session_id: session.id, action: action }),
  });
  if (res.ok) {
    const data = await res.json();
    currentPage = data.page;
    totalPages = data.total_pages;
    session.nativeWidth = data.width || session.nativeWidth;
    session.nativeHeight = data.height || session.nativeHeight;
    $("#pageCounter").textContent = `${currentPage} / ${totalPages}`;
    loadPreview();
    ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
    cancelPreview();
  }
};

$("#btnAutoMode").onclick = () => switchMode("auto");
$("#btnManualMode").onclick = () => switchMode("manual");

function switchMode(mode) {
  currentMode = mode;
  $$(".mode-btn").forEach((b) => b.classList.remove("active"));
  if (mode === "auto") {
    $("#btnAutoMode").classList.add("active");
    $("#autoUI").classList.remove("hidden");
    $("#manualUI").classList.add("hidden");
    drawCanvas.style.pointerEvents = "none";
    ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
    setBackground("bg_img_1.jpg");
  } else {
    $("#btnManualMode").classList.add("active");
    $("#manualUI").classList.remove("hidden");
    $("#autoUI").classList.add("hidden");
    drawCanvas.style.pointerEvents = "auto";
    setBackground("bg_img_2.jpg");
  }
}

window.setManualTab = function (type) {
  manualType = type;
  $$(".manual-tab").forEach((b) => b.classList.remove("active"));
  if (type === "text") {
    $("#tabText").classList.add("active");
    $("#manualTextInput").classList.remove("hidden");
    $("#manualImageInput").classList.add("hidden");
    $("#previewEditBtn").textContent = "1. Preview Text Replace";
  } else {
    $("#tabImage").classList.add("active");
    $("#manualImageInput").classList.remove("hidden");
    $("#manualTextInput").classList.add("hidden");
    $("#previewEditBtn").textContent = "1. Preview Image Replace";
  }
};

$("#manualReplaceImage").addEventListener("change", (e) => {
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (ev) => {
    manualReplaceImageBase64 = ev.target.result;
    $("#imageSelectName").textContent = file.name;
  };
  reader.readAsDataURL(file);
});

function clearFile() {
  session.id = null;
  fileInput.value = "";
  uploadSection.classList.remove("hidden");
  editSection.classList.add("hidden");
  canvasViewport.classList.add("hidden");
  emptyState.classList.remove("hidden");
  newEditBtn.classList.add("hidden");
  ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
  cancelPreview();
  resetZoom();
}

fileInput.addEventListener("change", async (e) => {
  const file = e.target.files[0];
  if (!file) return;

  const fd = new FormData();
  fd.append("file", file);
  const res = await fetch("/api/upload", { method: "POST", body: fd });
  const data = await res.json();

  session.id = data.session_id;
  session.nativeWidth = data.width;
  session.nativeHeight = data.height;
  currentPage = data.current_page;
  totalPages = data.total_pages;

  $("#pageCounter").textContent = `${currentPage} / ${totalPages}`;

  uploadSection.classList.add("hidden");
  editSection.classList.remove("hidden");
  emptyState.classList.add("hidden");
  canvasViewport.classList.remove("hidden");
  newEditBtn.classList.remove("hidden");

  updateHistoryButtons(false, false);
  loadPreview();
});

function loadPreview() {
  previewImg.src = `/api/preview/${session.id}?t=${Date.now()}`;
  previewImg.onload = () => {
    drawCanvas.width = previewImg.clientWidth;
    drawCanvas.height = previewImg.clientHeight;
    resetZoom();
  };
}

const ctx = drawCanvas.getContext("2d");
let isDrawing = false,
  startX,
  startY,
  currentX,
  currentY;

function getCoords(clientX, clientY) {
  const rect = drawCanvas.getBoundingClientRect();
  const scaleX = drawCanvas.width / rect.width;
  const scaleY = drawCanvas.height / rect.height;
  return {
    x: (clientX - rect.left) * scaleX,
    y: (clientY - rect.top) * scaleY,
  };
}

function startDrawing(clientX, clientY) {
  if (currentMode !== "manual") return;
  const pos = getCoords(clientX, clientY);
  startX = pos.x;
  startY = pos.y;
  isDrawing = true;
}

function drawLine(clientX, clientY) {
  if (!isDrawing) return;
  const pos = getCoords(clientX, clientY);
  currentX = pos.x;
  currentY = pos.y;
  ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
  ctx.strokeStyle = "#ff3366";
  ctx.lineWidth = 2;
  ctx.fillStyle = "rgba(255, 51, 102, 0.2)";
  ctx.fillRect(startX, startY, currentX - startX, currentY - startY);
  ctx.strokeRect(startX, startY, currentX - startX, currentY - startY);
}

async function stopDrawing() {
  if (!isDrawing) return;
  isDrawing = false;
  const scaleXNative = session.nativeWidth / drawCanvas.width;
  const scaleYNative = session.nativeHeight / drawCanvas.height;

  manualBoxNative = [
    Math.min(startX, currentX) * scaleXNative,
    Math.min(startY, currentY) * scaleYNative,
    Math.max(startX, currentX) * scaleXNative,
    Math.max(startY, currentY) * scaleYNative,
  ];

  $("#manualForm").classList.remove("hidden");
  $("#nudgeBlock").classList.remove("hidden");
  $("#scanResults").textContent = "Scanning formatting...";

  const res = await fetch("/api/extract", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ session_id: session.id, box: manualBoxNative }),
  });
  const data = await res.json();

  suggestedFontSize = data.suggested_font_size || 12;
  manualTextProps.angle = data.angle || 0;
  manualTextProps.is_bold = data.is_bold || false;
  manualTextProps.color = data.color || [0, 0, 0];

  const toHex = (c) => {
    let hex = Math.max(0, Math.min(255, Math.round(c * 255))).toString(16);
    return hex.length === 1 ? "0" + hex : hex;
  };
  const hexColor = `#${toHex(manualTextProps.color[0])}${toHex(manualTextProps.color[1])}${toHex(manualTextProps.color[2])}`;

  $("#manualTextColor").value = hexColor;
  $("#originalColorSwatch").dataset.color = hexColor;
  $("#originalColorSwatch").style.background = hexColor;

  let words = data.lines.reduce((acc, l) => acc + l.word_count, 0);
  $("#scanResults").innerHTML =
    `<b>Box Linked.</b> Original text detected as: ${manualTextProps.angle}° Rotation ${manualTextProps.is_bold ? "(Bold)" : ""}.<br><small>${data.lines.map((l) => l.text).join(" ")}</small>`;
}

drawCanvas.addEventListener("mousedown", (e) =>
  startDrawing(e.clientX, e.clientY),
);
drawCanvas.addEventListener("mousemove", (e) => drawLine(e.clientX, e.clientY));
drawCanvas.addEventListener("mouseup", stopDrawing);
drawCanvas.addEventListener("mouseleave", () => {
  if (isDrawing) stopDrawing();
});

drawCanvas.addEventListener(
  "touchstart",
  (e) => {
    if (e.touches.length === 1) {
      e.preventDefault();
      startDrawing(e.touches[0].clientX, e.touches[0].clientY);
    }
  },
  { passive: false },
);

drawCanvas.addEventListener(
  "touchmove",
  (e) => {
    if (e.touches.length === 1) {
      e.preventDefault();
      drawLine(e.touches[0].clientX, e.touches[0].clientY);
    }
  },
  { passive: false },
);

drawCanvas.addEventListener("touchend", () => {
  if (isDrawing) stopDrawing();
});

function updateHistoryButtons(canUndo, canRedo) {
  undoBtn.disabled = !canUndo;
  redoBtn.disabled = !canRedo;
}

async function triggerHistory(action) {
  const res = await fetch("/api/history", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ session_id: session.id, action: action }),
  });
  if (res.ok) {
    const data = await res.json();
    updateHistoryButtons(data.can_undo, data.can_redo);
    loadPreview();
    ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
  }
}

undoBtn.onclick = () => triggerHistory("undo");
redoBtn.onclick = () => triggerHistory("redo");

window.nudgePos = function (dx, dy) {
  const xInput = $("#manualOffsetX"),
    yInput = $("#manualOffsetY");
  xInput.value = parseFloat(xInput.value || 0) + dx;
  yInput.value = parseFloat(yInput.value || 0) + dy;
  submitEdit("manual", true);
};

window.nudgeSize = function (ds) {
  const sizeInput = $("#manualOffsetSize");
  sizeInput.value = parseFloat(sizeInput.value || 0) + ds;
  submitEdit("manual", true);
};

let boldLevel = 0;
window.nudgeBold = function (db) {
  boldLevel = Math.max(-4, Math.min(4, boldLevel + db));
  $("#manualOffsetBold").value = boldLevel;
  $("#boldLevelDisplay").textContent =
    `Weight: ${boldLevel === 0 ? "Normal" : boldLevel > 0 ? `B+${boldLevel}` : `B${boldLevel}`}`;
  submitEdit("manual", true);
};

window.setColor = function (hex) {
  $("#manualTextColor").value = hex;
  submitEdit("manual", true);
};

$("#manualTextColor").addEventListener("input", () =>
  submitEdit("manual", true),
);

window.cancelPreview = function () {
  loadPreview();
  $("#manualForm").classList.add("hidden");
  $("#nudgeBlock").classList.add("hidden");
  $("#manualSearchInput").value = "";
  $("#manualReplaceInput").value = "";
  $("#manualOffsetX").value = "0";
  $("#manualOffsetY").value = "0";
  $("#manualOffsetSize").value = "0";
  $("#manualOffsetBold").value = "0";
  boldLevel = 0;
  $("#boldLevelDisplay").textContent = `Weight: Normal`;
  manualReplaceImageBase64 = null;
  $("#imageSelectName").textContent = "Choose Image...";
  manualBoxNative = null;
  ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
};

window.submitEdit = async function (mode, isPreview = false) {
  let edits = [];

  if (mode === "auto") {
    edits = [...$$(".edit-row")]
      .map((row) => ({
        search: row.querySelector(".search-input").value.trim(),
        replace: row.querySelector(".replace-input").value,
      }))
      .filter((x) => x.search);
  } else if (mode === "manual") {
    if (!manualBoxNative)
      return showCustomAlert("Draw a box first.", "Selection Required");

    const hex = $("#manualTextColor").value;
    const customColor = [
      parseInt(hex.substring(1, 3), 16) / 255,
      parseInt(hex.substring(3, 5), 16) / 255,
      parseInt(hex.substring(5, 7), 16) / 255,
    ];

    if (manualType === "text") {
      edits = [
        {
          type: "text",
          box: manualBoxNative,
          search: $("#manualSearchInput").value,
          replace: $("#manualReplaceInput").value,
          offset_x: parseFloat($("#manualOffsetX").value) || 0,
          offset_y: parseFloat($("#manualOffsetY").value) || 0,
          offset_size: parseFloat($("#manualOffsetSize").value) || 0,
          offset_bold: parseInt($("#manualOffsetBold").value) || 0,
          font_size: suggestedFontSize,
          angle: manualTextProps.angle,
          is_bold: manualTextProps.is_bold,
          color: customColor,
        },
      ];
    } else {
      if (!manualReplaceImageBase64)
        return showCustomAlert(
          "Please select an image first.",
          "Image Required",
        );
      edits = [
        {
          type: "image",
          box: manualBoxNative,
          replace_image: manualReplaceImageBase64,
          offset_x: parseFloat($("#manualOffsetX").value) || 0,
          offset_y: parseFloat($("#manualOffsetY").value) || 0,
          offset_size: parseFloat($("#manualOffsetSize").value) || 0,
        },
      ];
    }
  }

  const res = await fetch("/api/edit", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      session_id: session.id,
      mode: mode,
      edits: edits,
      is_preview: isPreview,
    }),
  });

  if (res.ok) {
    const data = await res.json();
    if (data.status === "preview") {
      $("#previewImg").src = "data:image/png;base64," + data.image;
      $("#previewEditBtn").classList.add("hidden");
      ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
    } else {
      updateHistoryButtons(data.can_undo, data.can_redo);
      loadPreview();
      ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
      $("#manualForm").classList.add("hidden");
      $("#nudgeBlock").classList.add("hidden");
      $("#previewEditBtn").classList.remove("hidden");
      $("#manualSearchInput").value = "";
      $("#manualReplaceInput").value = "";
      $("#manualOffsetX").value = "0";
      $("#manualOffsetY").value = "0";
      $("#manualOffsetSize").value = "0";
      $("#manualOffsetBold").value = "0";
      boldLevel = 0;
      $("#boldLevelDisplay").textContent = `Weight: Normal`;
      manualReplaceImageBase64 = null;
      $("#imageSelectName").textContent = "Choose Image...";
      manualBoxNative = null;
      showToast("Changes Applied Successfully!");
    }
  } else {
    showCustomAlert("Editing failed. Please try again.", "Error");
  }
};

function downloadFile() {
  window.location.href = `/api/download/${session.id}`;
}

function addEditRow() {
  const row = document.createElement("div");
  row.className = "edit-row slide-in";
  row.draggable = true;
  row.innerHTML = `
    <div class="drag-handle">⠿</div>
    <div class="edit-fields">
      <input class="search-input" type="text" placeholder="Find">
      <button class="swap-btn" type="button">⇄</button>
      <input class="replace-input" type="text" placeholder="Replace">
    </div>
    <button class="remove-btn" type="button">×</button>
  `;
  row.querySelector(".remove-btn").onclick = () => row.remove();
  row.querySelector(".swap-btn").onclick = () => {
    const sInput = row.querySelector(".search-input"),
      rInput = row.querySelector(".replace-input"),
      temp = sInput.value;
    sInput.value = rInput.value;
    rInput.value = temp;
  };
  row.addEventListener("dragstart", () => row.classList.add("dragging-row"));
  row.addEventListener("dragend", () => row.classList.remove("dragging-row"));
  $("#edits").appendChild(row);
}

$("#edits").addEventListener("dragover", (e) => {
  e.preventDefault();
  const afterElement = [
    ...$("#edits").querySelectorAll(".edit-row:not(.dragging-row)"),
  ].reduce(
    (closest, child) => {
      const box = child.getBoundingClientRect();
      const offset = e.clientY - box.top - box.height / 2;
      return offset < 0 && offset > closest.offset
        ? { offset: offset, element: child }
        : closest;
    },
    { offset: Number.NEGATIVE_INFINITY },
  ).element;

  const draggable = document.querySelector(".dragging-row");
  if (draggable)
    afterElement == null
      ? $("#edits").appendChild(draggable)
      : $("#edits").insertBefore(draggable, afterElement);
});
$("#addBtn").onclick = addEditRow;
addEditRow();

function openAdmin() {
  $("#pinModal").classList.remove("hidden");
  $("#pinInput").value = "";
  setTimeout(() => $("#pinInput").focus(), 100);
}
function closePinModal() {
  $("#pinModal").classList.add("hidden");
}

// Auto-submit when 4 digits are entered
$("#pinInput").addEventListener("input", (e) => {
  if (e.target.value.length === 4) submitPin();
});
// Keep Enter key fallback
$("#pinInput").addEventListener("keypress", (e) => {
  if (e.key === "Enter") submitPin();
});

async function submitPin() {
  const pin = $("#pinInput").value;
  if (!pin) return;
  const btn = $("#submitPinBtn");
  const origText = btn.textContent;
  btn.textContent = "Verifying...";
  btn.disabled = true;

  try {
    const res = await fetch("/api/admin/verify", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ pin: pin }),
    });

    if (res.ok) {
      closePinModal();
      $("#adminModal").classList.remove("hidden");
      setBackground("bg_img_3.jpg");
      await loadAdminTab("Uploaded", "Images");
    } else {
      const content = $(".pin-content");
      content.classList.remove("shake");
      void content.offsetWidth;
      content.classList.add("shake");
      $("#pinInput").value = "";
      $("#pinInput").focus();
      showCustomAlert("Incorrect PIN. Please try again.", "Access Denied");
    }
  } catch (err) {
    showCustomAlert("Network error. Could not verify PIN.", "Error");
  } finally {
    btn.textContent = origText;
    btn.disabled = false;
  }
}

let vaultUnlocked = false;

window.toggleTrashView = function () {
  if (vaultUnlocked) {
    // If unlocked, clicking the icon hides the tabs and locks the vault again
    vaultUnlocked = false;
    $$(".trash-tab").forEach((tab) => tab.classList.add("hidden"));
    if (adminState.folder === "Deleted") loadAdminTab("Uploaded", "Images");
  } else {
    // If locked, prompt for PIN immediately upon clicking the 🐦‍🔥 icon
    $("#vaultPinModal").classList.remove("hidden");
    $("#vaultPinInput").value = "";
    setTimeout(() => $("#vaultPinInput").focus(), 100);
  }
};

function closeAdmin() {
  fetch("/api/admin/logout", { method: "POST" });
  $("#adminModal").classList.add("hidden");
  $$(".trash-tab").forEach((tab) => tab.classList.add("hidden"));
  vaultUnlocked = false; // Lock vault completely on exit
  setBackground(currentMode === "auto" ? "bg_img_1.jpg" : "bg_img_2.jpg");
  if (adminState.folder === "Deleted") loadAdminTab("Uploaded", "Images");
}

function closeVaultPinModal() {
  $("#vaultPinModal").classList.add("hidden");
}

// Auto-submit Vault PIN instantly when 4 digits are entered
$("#vaultPinInput").addEventListener("input", (e) => {
  if (e.target.value.length === 4) submitVaultPin();
});
$("#vaultPinInput").addEventListener("keypress", (e) => {
  if (e.key === "Enter") submitVaultPin();
});

async function submitVaultPin() {
  const pin = $("#vaultPinInput").value;
  if (!pin) return;
  const btn = $("#submitVaultPinBtn");
  const origText = btn.textContent;
  btn.textContent = "Verifying...";
  btn.disabled = true;

  try {
    const res = await fetch("/api/admin/verify_vault", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ pin: pin }),
    });
    if (res.ok) {
      vaultUnlocked = true;
      closeVaultPinModal();
      // Only reveal the hidden Deleted tabs upon successful PIN
      $$(".trash-tab").forEach((tab) => tab.classList.remove("hidden"));
    } else {
      const content = $("#vaultPinModal .pin-content");
      content.classList.remove("shake");
      void content.offsetWidth;
      content.classList.add("shake");
      $("#vaultPinInput").value = "";
      $("#vaultPinInput").focus();
      showCustomAlert(
        "Incorrect Vault PIN. Please try again.",
        "Access Denied",
      );
    }
  } catch (err) {
    showCustomAlert("Network error.", "Error");
  } finally {
    btn.textContent = origText;
    btn.disabled = false;
  }
}

window.loadAdminTab = async function (folder, type) {
  // Relock Vault automatically if user clicks away to normal Uploaded/Downloaded tabs
  if (folder !== "Deleted") {
    vaultUnlocked = false;
    $$(".trash-tab").forEach((tab) => tab.classList.add("hidden"));
  }

  adminState = { folder, type };

  $$(".admin-tab").forEach((b) => {
    b.classList.remove("active");
    if (b.textContent.trim() === `${folder} ${type}`) b.classList.add("active");
  });

  if (folder === "Deleted") {
    $("#adminTrashBtn").classList.add("hidden");
    $("#adminPermDeleteBtn").classList.remove("hidden");
    $("#adminRestoreBtn").classList.remove("hidden");
  } else {
    $("#adminTrashBtn").classList.remove("hidden");
    $("#adminPermDeleteBtn").classList.add("hidden");
    $("#adminRestoreBtn").classList.add("hidden");
  }

  $("#adminGrid").innerHTML = "";
  $("#adminLoader").classList.remove("hidden");
  $("#sortSelect").value = "date-desc"; // Reset Sort

  try {
    const res = await fetch(`/api/admin/list?folder=${folder}&type=${type}`);
    if (res.status === 401 || res.status === 403) {
      $("#adminLoader").classList.add("hidden");
      return showCustomAlert("Unauthorized access.", "Access Denied");
    }

    const files = await res.json();
    $("#adminLoader").classList.add("hidden");
    if (files.error)
      return showCustomAlert("Admin API Error: " + files.error, "API Error");

    currentAdminFiles = files; // Store files to allow sorting locally
    window.renderAdminGrid(); // Render with sorting
  } catch (error) {
    console.error("Dashboard Error:", error);
    $("#adminLoader").classList.add("hidden");
  }
};

window.renderAdminGrid = function () {
  const grid = $("#adminGrid");
  if (!currentAdminFiles || currentAdminFiles.length === 0) {
    grid.innerHTML =
      "<div class='empty-admin'>No files found in this directory.</div>";
    return;
  }

  const sortType = $("#sortSelect").value;

  // Regex finds any 14 digit sequence (YYYYMMDD_HHMMSS) anywhere in the name
  const getTimestamp = (name) => {
    const match = name.match(/(\d{8}_\d{6})/);
    return match ? match[1] : "";
  };

  let sortedFiles = [...currentAdminFiles];
  sortedFiles.sort((a, b) => {
    if (sortType === "name-asc") return a.name.localeCompare(b.name);
    if (sortType === "name-desc") return b.name.localeCompare(a.name);

    const timeA = getTimestamp(a.name);
    const timeB = getTimestamp(b.name);

    if (sortType === "date-desc") {
      if (timeA && timeB) return timeB.localeCompare(timeA);
      return b.name.localeCompare(a.name); // Fallback if no timestamp
    }
    if (sortType === "date-asc") {
      if (timeA && timeB) return timeA.localeCompare(timeB);
      return a.name.localeCompare(b.name); // Fallback if no timestamp
    }
    return 0;
  });

  grid.innerHTML = "";
  const isImg = adminState.type === "Images";

  sortedFiles.forEach((f) => {
    const item = document.createElement("div");
    item.className = "admin-file-card fade-in";
    item.innerHTML = `
      <input type="checkbox" class="admin-chk" data-path="${f.path}" data-sha="${f.sha}" data-url="${f.url}" data-name="${f.name}">
      <div class="file-visual">
          ${isImg ? `<img src="${f.url}" loading="lazy">` : `<div class="pdf-icon">PDF</div>`}
          <div class="file-actions">
              <button class="action-hover-btn" onclick="previewFile(event, '${f.url}', ${isImg}, '${f.name}')" title="Preview File">👁️</button>
              <button class="action-hover-btn" onclick="downloadSingleFile(event, '${f.url}', '${f.name}')" title="Download File">⬇️</button>
          </div>
      </div>
      <div class="file-name" title="${f.name}">${f.name}</div>
    `;
    item.onclick = (e) => {
      if (
        e.target.tagName !== "INPUT" &&
        !e.target.classList.contains("action-hover-btn")
      ) {
        const chk = item.querySelector("input");
        chk.checked = !chk.checked;
      }
    };
    grid.appendChild(item);
  });
};

window.downloadSingleFile = async function (e, url, name) {
  e.stopPropagation();
  try {
    const response = await fetch(url);
    if (!response.ok) throw new Error("Network response was not ok");
    const blob = await response.blob();
    const blobUrl = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.style.display = "none";
    a.href = blobUrl;
    a.download = name;
    document.body.appendChild(a);
    a.click();
    window.URL.revokeObjectURL(blobUrl);
    document.body.removeChild(a);
  } catch (err) {
    showCustomAlert("Failed to download file.", "Download Error");
  }
};

window.downloadSelected = async function () {
  const selected = [...$$(".admin-chk:checked")];
  if (selected.length === 0)
    return showCustomAlert(
      "Select at least one file to download.",
      "Selection Required",
    );
  $("#adminLoader").classList.remove("hidden");
  for (let i = 0; i < selected.length; i++) {
    await downloadSingleFile(
      { stopPropagation: () => {} },
      selected[i].dataset.url,
      selected[i].dataset.name,
    );
    await new Promise((r) => setTimeout(r, 500));
  }
  $("#adminLoader").classList.add("hidden");
  showToast("Downloads complete.");
};

window.previewFile = async function (e, url, isImg, name) {
  e.stopPropagation();
  setBackground("bg_img_4.jpg");
  $("#previewTitle").textContent = name;
  $("#filePreviewModal").classList.remove("hidden");
  const container = $("#previewContainer");

  if (isImg) {
    container.innerHTML = `<img src="${url}" style="max-width: 100%; max-height: 100%; object-fit: contain; border-radius: 8px;">`;
  } else {
    container.innerHTML = '<div class="loader"></div>';
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error("Failed to load");
      const rawBlob = await response.blob();
      const pdfBlob = new Blob([rawBlob], { type: "application/pdf" });
      if (currentBlobUrl) URL.revokeObjectURL(currentBlobUrl);
      currentBlobUrl = URL.createObjectURL(pdfBlob);
      container.innerHTML = `<iframe src="${currentBlobUrl}#toolbar=0&navpanes=0" style="width: 100%; height: 100%; border: none; border-radius: 8px; background: white;"></iframe>`;
    } catch (err) {
      container.innerHTML = `<div style="text-align: center; color: #94a3b8;">
        <div style="font-size: 40px; margin-bottom: 10px;">📄</div>
        <p>Browser preview unavailable.</p>
        <a href="${url}" target="_blank" class="primary-btn" style="text-decoration: none; display: inline-block; padding: 10px 20px; margin-top: 15px;">Download PDF Directly</a>
      </div>`;
    }
  }
};

window.closeFilePreview = function () {
  $("#filePreviewModal").classList.add("hidden");
  $("#previewContainer").innerHTML = "";
  if (currentBlobUrl) {
    URL.revokeObjectURL(currentBlobUrl);
    currentBlobUrl = null;
  }
  setBackground(
    $("#adminModal").classList.contains("hidden")
      ? currentMode === "auto"
        ? "bg_img_1.jpg"
        : "bg_img_2.jpg"
      : "bg_img_3.jpg",
  );
};

window.selectAll = function (state) {
  $$(".admin-chk").forEach((chk) => (chk.checked = state));
};

window.executeAdminAction = async function (action) {
  const selected = [...$$(".admin-chk:checked")].map((chk) => ({
    path: chk.dataset.path,
    sha: chk.dataset.sha,
  }));
  if (selected.length === 0)
    return showCustomAlert("Select at least one file.", "Selection Required");

  if (action === "delete") {
    const confirmed = await new Promise((resolve) => {
      $("#confirmModal").classList.remove("hidden");
      $("#confirmYesBtn").onclick = () => {
        $("#confirmModal").classList.add("hidden");
        resolve(true);
      };
      $("#confirmCancelBtn").onclick = () => {
        $("#confirmModal").classList.add("hidden");
        resolve(false);
      };
    });
    if (!confirmed) return;
  }

  $("#adminLoader").classList.remove("hidden");
  try {
    const res = await fetch("/api/admin/action", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ action: action, files: selected }),
    });
    if (res.ok) await loadAdminTab(adminState.folder, adminState.type);
    else {
      showCustomAlert("Action failed or unauthorized.", "Action Failed");
      $("#adminLoader").classList.add("hidden");
    }
  } catch (error) {
    console.error("Action Error:", error);
    showCustomAlert("Network error occurred.", "Network Error");
    $("#adminLoader").classList.add("hidden");
  }
};

root@dc:~/srinu/pdf_edit# 
