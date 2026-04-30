# Central Spark Psychotherapy — Deployment

## Universal init

On `/init`, follow `../../CONTROL/INIT_PROTOCOL.md`. This repo file adds local rules only; it does not own the workspace init protocol.

---

Static single-file HTML site for Careena Cook / centralsparkpsychotherapy.com.au.
No build step. No framework. One file: `index.html`.

---

## What to do

1. **Create GitHub repo** `ethikslabs/central-spark` and push this folder
2. **Deploy to EC2** at `/home/ec2-user/sites/central-spark/`
3. **Create Nginx vhost** for `centralpark.ethikslabs.com`
4. **Add Cloudflare DNS** A record pointing at EC2
5. **Smoke test** the live URL

Full detail below.

---

## Infrastructure

```
EC2 IP:     54.252.185.132
EC2 user:   ec2-user
Region:     ap-southeast-2 (Sydney)
DNS:        Cloudflare → ethikslabs.com
SSL:        Cloudflare Flexible (no Certbot needed)
Nginx:      Already running other vhosts
```

---

## 1 — GitHub

```bash
git init
git remote add origin git@github.com:ethikslabs/central-spark.git
git add .
git commit -m "initial: central spark staging site"
git push -u origin main
```

---

## 2 — EC2 deploy

```bash
ssh ec2-user@54.252.185.132

mkdir -p /home/ec2-user/sites
cd /home/ec2-user/sites
git clone git@github.com:ethikslabs/central-spark.git central-spark

chmod 644 /home/ec2-user/sites/central-spark/index.html
chmod 755 /home/ec2-user/sites/central-spark
```

---

## 3 — Nginx vhost

Check where vhosts live first (`/etc/nginx/conf.d/` or `/etc/nginx/sites-available/`), then create:

```nginx
server {
    listen 80;
    server_name centralpark.ethikslabs.com;

    root /home/ec2-user/sites/central-spark;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    gzip on;
    gzip_types text/html text/css application/javascript;
}
```

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 4 — Cloudflare DNS

Add A record in Cloudflare → ethikslabs.com:

```
Type:   A
Name:   centralpark
Value:  54.252.185.132
Proxy:  ON (orange cloud)
TTL:    Auto
```

---

## 5 — Smoke test

- [ ] https://centralpark.ethikslabs.com loads
- [ ] All 6 nav pages work (Home, Psychotherapy, EMDR, About, FAQ, Contact)
- [ ] Careena's photo loads on About
- [ ] Google Maps loads on Home, About, Contact
- [ ] Mobile burger menu works
- [ ] No console errors

---

## Future updates

```bash
# Mac
git add index.html && git commit -m "update: ..." && git push

# EC2
cd /home/ec2-user/sites/central-spark && git pull
# No nginx restart needed — static file serving
```

---

## Known TODOs for Careena

- **Contact form** — currently visual only. Wire up Formspree (free tier):
  get endpoint at formspree.io, replace `onsubmit="handleForm(event)"` with
  `action="https://formspree.io/f/XXXX" method="POST"` and remove the JS handler.

- **Headshot** — currently loaded from her WordPress CDN. If she ever migrates,
  save `image-48-rotated.jpg` locally into `/assets/` and update the `src`.
