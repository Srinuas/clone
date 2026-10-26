<!-- From this: -->
<link rel="icon" type="image/jpeg" href="assets/favicon.jpg">

<!-- To this: -->
<link rel="icon" type="image/jpeg" href="favicon.jpg">


let currentBg = 'bg.jpg';


// Replace any setBackground('assets/bg_img_X.jpg') or setBackground('bg_img_X.jpg') with:
setBackground('bg.jpg');


# 1. Stop the current container
docker rm -f app

# 2. Run with --env-file and -v (Volume Mount)
docker run -itd \
  --name app \
  --restart unless-stopped \
  --env-file .env \
  -p 333:3000 \
  -v "$PWD:/usr/src/app" \
  srinuas/pdfimg:1


  
