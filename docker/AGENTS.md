# Docker — Agent Kuralları

Bu dizin projenin container yapılandırmasını içerir.

## Servisler (hedef)

| Servis | Kaynak | Port | Açıklama |
|--------|--------|------|----------|
| `nginx` | `nginx/default.conf` | `8080:80` | Reverse proxy |
| `php` | `php/8.3/dockerfile` | — | PHP 8.3-FPM, Composer |
| `postgres` | `postgres:16` | `5432` | PostgreSQL |
| `mailpit` | `axllent/mailpit` | `8025`, `1025` | SMTP + web UI |
| `worker` | php image + supervisord | — | `queue:work` + `schedule:work` |

Volume: proje kökü `../` → container içi `/var/www/html`

> **Not:** Mevcut compose dosyası henüz eski yapıdadır (PHP 8.2 + Apache + MariaDB). Hedef yapı yukarıdaki tablodur; refactor sırasında güncellenecek.

## Mailpit (local e-posta test)

| Adres | Port | Açıklama |
|-------|------|----------|
| http://localhost:8025 | 8025 | Gelen kutusu web arayüzü |
| `mailpit:1025` (container içi) | 1025 | SMTP |

Laravel `.env`:

- **SMTP host:** `mailpit`
- **Port:** `1025`
- **User / pass:** boş bırakılabilir

Test: mail gönder → http://localhost:8025

## PostgreSQL (hedef)

- **Host (container içi):** `postgres`
- **Veritabanı:** `laravel_oauth`
- **User/Pass:** `docker-compose.yml` env değerlerinden

## Komutlar

Proje kökünden:

```bash
docker compose -f docker/docker-compose.yml up -d
docker compose -f docker/docker-compose.yml up --build -d
docker compose -f docker/docker-compose.yml exec php bash
docker compose -f docker/docker-compose.yml exec php composer install
docker compose -f docker/docker-compose.yml exec php php artisan migrate --seed
docker compose -f docker/docker-compose.yml logs -f worker
docker compose -f docker/docker-compose.yml ps
```

`docker/` dizininden:

```bash
docker compose up -d
docker compose exec php bash
docker compose logs -f
```

## Notlar

- Container adları sabit değildir; compose proje adına göre değişir (`COMPOSE_PROJECT_NAME` → `docker/.env`)
- `docker-compose` yerine `docker compose` (v2) kullanın
- PHP extension hedefi: `pdo_pgsql` (mysql değil)
- Worker ayrı container; web istekleri php-fpm üzerinden nginx'e gelir
- Büyük SQL dump dosyaları `.cursorignore` içinde — grep'e dahil edilmez
