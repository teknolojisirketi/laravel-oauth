# Docker — Agent Kuralları

Bu dizin projenin container yapılandırmasını içerir.

## Servisler

| Servis | Kaynak | Port | Açıklama |
|--------|--------|------|----------|
| `nginx` | `nginx/default.conf` | `8080:80` | Reverse proxy → php-fpm |
| `php` | `php/8.3/dockerfile` | — | PHP 8.3-FPM, Composer, pdo_pgsql |
| `postgres` | `postgres:16-alpine` | `5432:5432` | PostgreSQL |
| `mailpit` | `axllent/mailpit` | `8025`, `1025` | SMTP + web UI |
| `worker` | php image + supervisord | — | `queue:work` + `schedule:work` |

Volume: proje kökü `../` → container içi `/var/www/html`

## Ortam (`docker/.env`)

Compose değişkenleri `docker/.env` dosyasından okunur. Şablon: `docker/.env.example`.

```bash
cp docker/.env.example docker/.env
```

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `COMPOSE_PROJECT_NAME` | `laravel-oauth` | Container adı öneki |
| `DOCKER_NETWORK` | `app_network` | Gerçek Docker network adı (`networks.app_network.name`) |
| `DOCKER_NETWORK_DRIVER` | `bridge` | Network driver |
| `NGINX_PORT` | `8080` | Web portu |
| `POSTGRES_PORT` | `5432` | PostgreSQL host portu |
| `MAILPIT_UI_PORT` | `8025` | Mailpit web UI |
| `MAILPIT_SMTP_PORT` | `1025` | Mailpit SMTP |
| `POSTGRES_DB` | `laravel_oauth` | Veritabanı adı |
| `POSTGRES_USER` | `laravel` | DB kullanıcı |
| `POSTGRES_PASSWORD` | `secret` | DB şifre |

## Worker: tek container + supervisord (ilk plan — değişmez)

Jobs için **yalnızca bir** compose servisi: `worker`. Ayrı `consumer` / `scheduler` servisi **yok**.

Supervisord container içinde iki süreci yönetir:

| Program | Komut |
|---------|--------|
| `queue` | `php artisan queue:work --sleep=3 --tries=3` |
| `schedule` | `php artisan schedule:work` |

Config: `docker/worker/supervisord.conf`

### Local / staging / prod VPS (tek sunucu)

Staging ve production'da da **aynı `docker-compose.yml`** tek sunucuda çalışır:

```bash
docker compose -f docker/docker-compose.yml up -d
```

Tüm servisler (nginx, php, postgres, mailpit, **worker**) aynı makinede. Jobs tarafı için ek container veya host-level supervisord gerekmez — `worker` + supervisord yeterli.

Worker logları: `docker compose -f docker/docker-compose.yml logs -f worker`

## PostgreSQL

| Alan | Değer |
|------|-------|
| Host (container içi) | `postgres` |
| Host (WSL dışından) | `127.0.0.1:5432` |
| Veritabanı | `laravel_oauth` |
| Kullanıcı | `laravel` |
| Şifre | `secret` |

Laravel `.env`:

```
DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=laravel_oauth
DB_USERNAME=laravel
DB_PASSWORD=secret
```

## Mailpit (local e-posta test)

| Adres | Port | Açıklama |
|-------|------|----------|
| http://localhost:8025 | 8025 | Gelen kutusu web arayüzü |
| `mailpit:1025` (container içi) | 1025 | SMTP |

Laravel `.env`:

```
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

## Komutlar

Özet kurulum: proje kökündeki [`README.md`](../README.md).

Proje kökünden:

```bash
docker compose -f docker/docker-compose.yml up -d
docker compose -f docker/docker-compose.yml up --build -d
docker compose -f docker/docker-compose.yml exec php bash
docker compose -f docker/docker-compose.yml exec php composer install
docker compose -f docker/docker-compose.yml exec php php artisan key:generate
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

- Container adları compose proje adına göre değişir (`COMPOSE_PROJECT_NAME` → `docker/.env`)
- `docker compose` (v2) kullanın
- Web: http://localhost:8080 (Laravel kurulunca `public/index.php`)
- Worker, Laravel kurulana kadar `artisan` bulunamadığı için restart eder — normal
- `docker/.env` commit edilmez
- SQL dump dosyaları `.gitignore` / `.cursorignore` içinde
