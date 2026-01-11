# Customer Support Omnichannel Platform

Integrated Telegram & WhatsApp APIs with AI-powered chat analysis and message archiving

Internet
   ↓
Cloudflare (DNS / SSL / WAF)
   ↓
AWS EC2 Singapore（Docker）
   ├── Nginx
   ├── React (frontend)
   ├── Laravel (backend)
   ├── MySQL / RDS
   └── Redis / Kafka


  步骤 1：确认域名注册状态

登陆 Cloudflare → Registrar

找到你的 domain → 查看 Cloudflare 提供的 nameservers

检查是否是 Cloudflare 提供的两条 NS

类似 abby.ns.cloudflare.com / alex.ns.cloudflare.com

如果域名是 直接在 Cloudflare 买的 → 等待 24–48 小时

Cloudflare 会自动把 nameservers 激活

期间 不要改其他 DNS

⚠️ 不要手动去 registrar 改，因为你直接在 Cloudflare Registrar 买，后台会自己管理




步骤 2：准备服务器

在 AWS 新加坡开 EC2：

Ubuntu 22.04 LTS

t3.small

Key Pair（.pem）

Security Group：SSH 22 / HTTP 80 / HTTPS 443

SSH 登录 EC2：

ssh -i cslive.pem ubuntu@<EC2_PUBLIC_IP>



步骤 3：安装 Docker + Docker Compose
sudo apt update
sudo apt install -y ca-certificates curl gnupg
curl -fsSL https://get.docker.com | sudo sh
sudo apt install docker-compose-plugin -y
sudo usermod -aG docker $USER
exit


重新 SSH 登录生效

测试：

docker --version
docker compose version


步骤 4：部署 CSLive 项目

Clone 你的项目：

git clone <你的仓库地址>
cd cslive


配置 .env（backend）：

APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.cslive.io

DB_CONNECTION=mysql
DB_HOST=mysql
DB_DATABASE=cslive
DB_USERNAME=root
DB_PASSWORD=root

REDIS_HOST=redis
QUEUE_CONNECTION=redis


确认 Dockerfile 和 docker-compose.yml 正确（frontend + backend + nginx + mysql + redis）

启动 Docker：

docker compose up -d --build


Laravel 初始化：

docker compose exec backend php artisan key:generate
docker compose exec backend php artisan migrate

步骤 5：Cloudflare DNS 指向服务器

登录 Cloudflare → DNS

添加记录：

A → api.cslive.io → EC2 公网 IP

A → app.cslive.io → EC2 公网 IP

开启 SSL（Full Strict 推荐）

测试：

http://api.cslive.io
http://app.cslive.io


等待几分钟到 1 小时 DNS 生效

步骤 6：生产优化

React frontend build：

cd frontend
npm run build


nginx 指向 React dist

Laravel cache：

php artisan config:cache
php artisan route:cache
php artisan view:cache


Docker Compose 设置 restart: always
