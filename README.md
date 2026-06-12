# Laravel OAuth Server

Laravel + Passport tabanlı OAuth 2.0 / OpenID Connect authorization server.

Detaylı agent kuralları: [`AGENTS.md`](AGENTS.md) · Docker: [`docker/AGENTS.md`](docker/AGENTS.md)

## Gereksinimler

- Docker Desktop (WSL entegrasyonu açık)
- Git

## Hızlı başlangıç

```bash
cd ~/www/laravel-oauth

# İlk kurulum: Docker ortam dosyası
cp docker/.env.example docker/.env

# Container'ları build et ve başlat (env: docker/.env)
docker compose -f docker/docker-compose.yml up --build -d

# Durum kontrolü
docker compose -f docker/docker-compose.yml ps
```

| Adres | Açıklama |
|-------|----------|
| http://localhost:8080 | Web (nginx → php-fpm) |
| http://localhost:8025 | Mailpit (e-posta test UI) |

## Container içinde çalışma (bash)

PHP container'ına gir:

```bash
docker compose -f docker/docker-compose.yml exec php bash
```

Container içindeyken (`/var/www/html`):

```bash
composer install
php artisan key:generate
php artisan migrate
php artisan migrate --seed
php artisan route:list
```

Container'dan çıkmak: `exit`

## Tek satır komutlar (host'tan, bash açmadan)

```bash
docker compose -f docker/docker-compose.yml exec php composer install
docker compose -f docker/docker-compose.yml exec php php artisan key:generate
docker compose -f docker/docker-compose.yml exec php php artisan migrate
docker compose -f docker/docker-compose.yml exec php php artisan migrate --seed
```

## Loglar ve yeniden başlatma

```bash
# Tüm servisler
docker compose -f docker/docker-compose.yml logs -f

# Sadece worker (queue + schedule)
docker compose -f docker/docker-compose.yml logs -f worker

# Yeniden build
docker compose -f docker/docker-compose.yml up --build -d

# Durdur
docker compose -f docker/docker-compose.yml down
```

## Servisler

| Servis | Port | Açıklama |
|--------|------|----------|
| `nginx` | 8080 | Reverse proxy |
| `php` | — | PHP 8.3-FPM |
| `postgres` | 5432 | PostgreSQL 16 |
| `mailpit` | 8025, 1025 | SMTP test |
| `worker` | — | supervisord: `queue:work` + `schedule:work` |

## Ortam (.env)

Docker içinden çalışırken örnek değerler:

```env
APP_URL=http://localhost:8080
DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=laravel_oauth
DB_USERNAME=laravel
DB_PASSWORD=secret
QUEUE_CONNECTION=database
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
```

## `docker/` dizininden çalıştırma

```bash
cd docker
docker compose up --build -d
docker compose exec php bash
```
