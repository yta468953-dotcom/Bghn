name: CLOUDFLARE_SHADOW_VPS
on: workflow_dispatch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Initialize Shadow Core
        run: |
          # 1. إعداد المستخدم (Password: shadow123)
          sudo useradd -m shadow -s /bin/bash
          sudo usermod -aG sudo shadow
          echo "shadow:shadow123" | sudo chpasswd
          
          # 2. تثبيت Cloudflared (بدون Ngrok أو Tmate)
          wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
          sudo dpkg -i cloudflared.deb
          
          # 3. تشغيل الـ SSH
          sudo service ssh start
          
          # 4. فتح النفق وطباعة الرابط للـ Termius
          echo "LAUNCHING TUNNEL..."
          nohup cloudflared tunnel --url tcp://localhost:22 > cf_link.log 2>&1 &
          sleep 20
          echo "------------------------------------------------"
          echo ">>> COPY THIS LINK FOR TERMIUS HOSTNAME <<<"
          grep -oE "https://[a-zA-Z0-9.-]+\.trycloudflare\.com" cf_link.log | sed 's/https:\/\//tcp:\/\//'
          echo "------------------------------------------------"
          
          # إبقاء الجلسة نشطة (6 ساعات)
          sleep 21600
          
