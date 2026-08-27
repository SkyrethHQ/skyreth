<a style="text:center"># Skyreth Panel</a>

The modern client panel for Pterodactyl.

Skyreth sits on top of Pterodactyl. Ptero runs the servers. Skyreth runs the clients, the coins, the store, and the payments — Stripe, Tebex, and other gateways.

Website: skyreth.com
App login: panel.skyreth.com
Product page: skyreth.com/panel

--------------------------------------------------
What it does
--------------------------------------------------

- Resource management — create servers, gift them, assign RAM/CPU/disk
- Coins — earn on the AFK page, via Linkvertise, or gift to other users
- Renewals — servers stay up if the client pays in coins
- Coupons — drop resources and coins on a user
- Servers — create, view, and edit
- Payments — Stripe, Tebex, and other methods
- Login queue — stops the panel from getting slammed
- Users — auth, password reset, accounts
- Store — buy resources with coins
- Control panel — see what the user actually owns
- Join-for-rewards — Discord server join → coins
- Admin — set / add / remove coins and resources; create / revoke coupons
- API — for bots and anything else you wire in

Skyreth does not replace Pterodactyl. You need a working Ptero panel first.

--------------------------------------------------
Credit
--------------------------------------------------

Keep "Powered by Skyreth Panel" in the footer if you can. That’s how the project stays visible. Support is for installs that keep the footer. Removing it doesn’t make the software illegal — it just means you’re on your own.

--------------------------------------------------
Install
--------------------------------------------------

You need Pterodactyl already running.

Option A — Pterodactyl egg (Node)

1. Upload the panel files to a Pterodactyl Node.js server.
   Generic Node egg: https://github.com/parkervcp/eggs/blob/master/generic/nodejs/egg-node-js-generic.json
2. Unzip and set the server to the Node version Skyreth ships with (check the repo; don’t assume Node 16 forever).

Option B — Direct on a VPS

1. Node

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

Open a new SSH session, then:

nvm install 20
node -v

Use the version listed in the Skyreth repo if it differs.

2. Files

git clone https://github.com/YOURUSER/skyreth.git /var/www/skyreth

3. Build deps + modules

apt-get update && apt-get install libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev build-essential
cd /var/www/skyreth && npm i

After settings.json is filled in:

node index.js

Background:

screen -S skyreth node index.js

Detach: Ctrl + A + D
Reattach: screen -R skyreth
Stop: Ctrl + C

--------------------------------------------------
Web server
--------------------------------------------------

1. Edit settings.json — Pterodactyl panel URL + API key, Discord auth, Stripe, Tebex, other gateways.
2. Start Skyreth. Ignore the two weird boot warnings if they still show.
3. Point DNS at the VPS (panel.yourdomain.com → VPS IP).
4. On the VPS:

apt install nginx certbot
ufw allow 80
ufw allow 443
certbot certonly -d panel.yourdomain.com
nano /etc/nginx/sites-enabled/skyreth.conf

5. Paste the nginx block below. Swap domain, SSL paths, and the local Skyreth port.
6. systemctl restart nginx and open the domain.

Nginx:

server {
    listen 80;
    server_name panel.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name panel.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/panel.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/panel.yourdomain.com/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location /afkwspath {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://localhost:<port>/afkwspath;
    }

    location / {
        proxy_pass http://localhost:<port>/;
        proxy_buffering off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

--------------------------------------------------
PM2 (run on boot, not inside Ptero)
--------------------------------------------------

npm install pm2 -g
cd /var/www/skyreth
pm2 start index.js --name "Skyreth"
pm2 logs Skyreth
pm2 startup
pm2 save

Stop: pm2 stop Skyreth
Status: pm2 list
Disable startup: pm2 unstartup

--------------------------------------------------
Payments
--------------------------------------------------

Wire these in settings.json:

- Stripe — cards, direct checkout
- Tebex — game-store packages, Minecraft-friendly
- Other gateways you enable in config

Pterodactyl still creates/deletes the actual server. Skyreth just takes the money and tells Ptero what to spin up.
