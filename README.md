Universal AWS EC2 Deployment Guide (Any Web Application)
1. Launch EC2 Instance
- Go to AWS EC2 Dashboard
- Click Launch Instance
- Choose Ubuntu Server 22.04
- Select t2.micro (Free Tier)
- Create/Select Key Pair (.pem file)
- Allow Inbound Rules: SSH (22), HTTP (80), Custom (5000 if backend)
- Launch Instance
2. Connect to Server
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@PUBLIC_IP
3. Update Server
sudo apt update && sudo apt upgrade -y
4. Install Required Tools
sudo apt install git curl nginx -y
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
5. Clone Project
git clone <repo-url>
cd <project-folder>
6. Backend Setup (MERN)
cd backend
npm install
nano .env (add PORT, MONGO_URI etc.)
pm2 list
pm2 delete all
pm2 start server.js --name backend
pm2 save
pm2 startup
7. Frontend Setup
cd frontend
npm install
npm run build
sudo cp -r dist/* /var/www/html/ (or build/*)
8. Configure Nginx
sudo nano /etc/nginx/sites-available/default

server {
  listen 80;
  server_name _;

  root /var/www/html;
  index index.html;

  location / {
    try_files $uri /index.html;
  }

  location /api/ {
    proxy_pass http://localhost:5000;
    proxy_set_header Host $host;
  }
}

sudo nginx -t
sudo systemctl restart nginx
9. Common Fixes
- Ensure backend listens on 0.0.0.0
- Open required ports in Security Group
- Use pm2 logs for debugging
- Check nginx with: sudo nginx -t
10. Final URLs
http://PUBLIC_IP (Frontend)
http://PUBLIC_IP:5000 (Backend API)
