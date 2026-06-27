# BJJ74 — сайт секции грепплинга (bjj74.ru)

Статический сайт на **Hugo** (тема — собственные шаблоны в `layouts/`). Контент — в `content/`,
сборка кладёт готовую статику в `public/`. Полные правила контента и вёрстки — в `CLAUDE.md`
(и в воркбенче `~/projects/grappling-chelyabinsk-vault`).

## Локальная разработка

```bash
hugo server -D            # предпросмотр на http://localhost:1313/ (с черновиками)
hugo server               # без черновиков, как на проде
```

## Production-сборка

```bash
hugo --gc --minify        # результат — в public/
```

- `baseURL` задан в `hugo.yaml` (`https://bjj74.ru/`).
- `buildFuture: true` — посты с будущей датой тоже рендерятся (чтобы на хостинге появлялось всё сразу).
- **Скрыть страницу/пост** можно только через `draft: true` в frontmatter (тогда в проде её нет).
  Всё, что `draft: false` — попадает на боевой сайт.

---

## Деплой и хостинг

Сайт — это статика из `public/`. Любой из вариантов ниже **сам собирает Hugo при каждом push**
в GitHub-репозиторий `tym83-ai/bjj74-grappling`, делает **preview-сборку на каждый Pull Request**
(уникальный временный URL) и **обновляет прод при мердже в `main`**.

### Вариант 1 — Cloudflare Pages (рекомендую: бесплатно, приватный репо, превью PR, домен + SSL)

1. Dash Cloudflare → **Workers & Pages → Create → Pages → Connect to Git**.
2. Авторизовать Cloudflare в организации `tym83-ai`, выбрать репозиторий `bjj74-grappling`.
3. Настройки сборки:
   - **Framework preset:** Hugo
   - **Build command:** `hugo --gc --minify -b $CF_PAGES_URL`
     (для прод-домена Cloudflare сам подставит правильный URL; для PR — превью-URL, ссылки не «уедут» на bjj74.ru)
   - **Build output directory:** `public`
   - **Production branch:** `main`
   - **Environment variables:** `HUGO_VERSION = 0.162.1`, `HUGO_ENV = production`
4. Save and Deploy. Дальше каждый push в `main` → прод, каждый PR → отдельный preview-URL.

### Вариант 2 — Netlify

Положить в корень репо `netlify.toml`:

```toml
[build]
  command = "hugo --gc --minify -b $URL"
  publish = "public"
[build.environment]
  HUGO_VERSION = "0.162.1"
  HUGO_ENV = "production"
[context.deploy-preview]
  command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"   # корректные ссылки в превью PR
```

Netlify → Add new site → Import from Git → выбрать `bjj74-grappling`. Deploy previews на PR включены по умолчанию.

### Вариант 3 — GitHub Pages (через GitHub Actions)

Положить `.github/workflows/deploy.yml`:

```yaml
name: Deploy Hugo to Pages
on:
  push: { branches: [main] }
  workflow_dispatch:
permissions: { contents: read, pages: write, id-token: write }
concurrency: { group: pages, cancel-in-progress: true }
jobs:
  build:
    runs-on: ubuntu-latest
    env: { HUGO_VERSION: 0.162.1 }
    steps:
      - uses: actions/checkout@v4
        with: { submodules: recursive, fetch-depth: 0 }
      - run: |
          wget -qO hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i hugo.deb
      - uses: actions/configure-pages@v5
        id: pages
      - run: hugo --gc --minify -b "${{ steps.pages.outputs.base_url }}/"
      - uses: actions/upload-pages-artifact@v3
        with: { path: ./public }
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: { name: github-pages, url: "${{ steps.deployment.outputs.page_url }}" }
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

Затем Settings → Pages → Source = **GitHub Actions**.
⚠️ Pages для **приватного** репо доступен только на платных планах (GitHub Pro/Team). На бесплатном — либо сделать репо публичным, либо взять Вариант 1/2/4. Превью на PR из коробки нет.

### Вариант 4 — Свой VPS (nginx) + авто-деплой из GitHub Actions

На сервере: nginx отдаёт `/var/www/bjj74`, домен через certbot (Let's Encrypt).

`.github/workflows/deploy-vps.yml`:

```yaml
name: Build & rsync to VPS
on: { push: { branches: [main] } }
jobs:
  deploy:
    runs-on: ubuntu-latest
    env: { HUGO_VERSION: 0.162.1 }
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - run: |
          wget -qO hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i hugo.deb
      - run: hugo --gc --minify
      - uses: webfactory/ssh-agent@v0.9.0
        with: { ssh-private-key: "${{ secrets.DEPLOY_SSH_KEY }}" }
      - run: |
          rsync -az --delete public/ ${{ secrets.DEPLOY_USER }}@${{ secrets.DEPLOY_HOST }}:/var/www/bjj74/
```

Секреты репо (Settings → Secrets and variables → Actions): `DEPLOY_SSH_KEY`, `DEPLOY_USER`, `DEPLOY_HOST`.
nginx (фрагмент): `root /var/www/bjj74; index index.html; location / { try_files $uri $uri/ =404; }`, затем `sudo certbot --nginx -d bjj74.ru -d www.bjj74.ru`.

### Привязка домена bjj74.ru

- **Cloudflare Pages / домен на Cloudflare:** Pages → проект → **Custom domains → Set up a domain → `bjj74.ru`** (и `www`). DNS и бесплатный SSL Cloudflare настроит сам.
- **Домен у другого регистратора (CF Pages/Netlify):** в DNS у регистратора:
  - `www` → `CNAME` на `<project>.pages.dev` (CF) или `<site>.netlify.app` (Netlify);
  - корень `bjj74.ru` (apex) → `A`/`ALIAS` на адреса платформы (CF/Netlify дают их в разделе Custom domains).
- **GitHub Pages:** Settings → Pages → Custom domain = `bjj74.ru`; в DNS — `A` на адреса GitHub Pages (185.199.108–111.153) + `www` `CNAME` на `tym83-ai.github.io`.
- **Свой VPS:** `A`-запись `bjj74.ru` → IP сервера, `www` `CNAME` → `bjj74.ru`; SSL через certbot.

### Как это работает с PR

- **Новый PR** → платформа (CF Pages/Netlify) собирает сайт и даёт **отдельный временный URL** для проверки.
- **Мердж в `main`** → пересборка и **обновление боевого сайта** автоматически.
- Скрытые материалы — это `draft: true`; `draft: false` сразу попадает в сборку (и в превью, и в прод).

## Доступы

- Репозиторий: `https://github.com/tym83-ai/bjj74-grappling` (private). Деплой/настройки — у админов репо.
