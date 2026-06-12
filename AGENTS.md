# Laravel OAuth Server — Agent Kuralları

Cursor ve diğer AI ajanları için proje referansı. Yapı ve kurallar değişirse bu dosyayı güncelleyin.

## Proje Özeti

Laravel + Passport tabanlı OAuth 2.0 / OpenID Connect authorization server.
PostgreSQL, Docker (nginx, php-fpm, mailpit, worker). Admin paneli Tabler + Blade + native JS (npm yok).

## Tech Stack

- **Framework:** Laravel 11+
- **PHP:** 8.3 (php-fpm)
- **Veritabanı:** PostgreSQL 16
- **OAuth:** Laravel Passport (OIDC, PKCE, client credentials, refresh token rotation)
- **RBAC:** spatie/laravel-permission (resource + own/all scope)
- **2FA:** pragmarx/google2fa-laravel (TOTP + e-posta OTP yedek)
- **Admin UI:** Tabler (statik, `public/admin/`) + Blade
- **Frontend JS:** Native ES modules (`public/js/admin/`) — npm/Vite/React/Vue/Alpine YOK
- **Mail (local):** Mailpit
- **Queue:** database driver (Redis yok)

## Docker

Detaylar için [`docker/AGENTS.md`](docker/AGENTS.md).

```bash
docker compose -f docker/docker-compose.yml up -d
docker compose -f docker/docker-compose.yml exec php bash
docker compose -f docker/docker-compose.yml exec php composer install
docker compose -f docker/docker-compose.yml exec php php artisan migrate
```

## Klasör Yapısı

```
laravel-oauth/
├── AGENTS.md
├── .cursorignore
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Admin/       # administrators, users, clients, roles, permissions, scopes
│   │   │   └── Auth/        # login, 2fa, password
│   │   └── Middleware/
│   │       ├── EnsureAdmin.php
│   │       └── EnsureTwoFactor.php
│   ├── Models/
│   │   └── User.php
│   └── Support/
│       └── TwoFactor.php
├── docker/                  # → docker/AGENTS.md
├── public/
│   ├── admin/               # Tabler dist (css, js, fonts)
│   └── js/admin/            # app.js, api.js, confirm.js, toast.js
├── resources/views/
│   ├── layouts/admin.blade.php
│   ├── auth/                # login, 2fa-challenge, 2fa-setup
│   ├── admin/               # CRUD sayfaları
│   └── oauth/               # consent ekranı
└── routes/
    ├── web.php              # admin + auth
    └── api.php              # Passport korumalı API
```

## Kod Kuralları (sade yapı)

### Yapılacak

- İnce controller'lar; validation: `$request->validate()` controller içinde
- Tek `User` modeli; rol ayrımı Spatie ile (`admin`, `user`)
- Custom middleware: sadece `EnsureAdmin`, `EnsureTwoFactor`
- Spatie `permission:` middleware route tanımında
- own/all filtreleme controller içinde basit if
- Formlar çoğunlukla klasik POST + Blade (JS gerektirmez)

### Yapılmayacak

- `app/Services/`, `app/Repositories/`, `app/Actions/` katmanları
- Event/Listener (gerekmedikçe)
- Policy sınıfları (başlangıçta)
- FormRequest (tekrar eden validation hariç)
- npm, Vite, axios, jQuery, React, Vue, Alpine, Livewire, Filament, Fortify, Sanctum, Redis

## Kullanıcı Tipleri

| Tip | Rol | Giriş | 2FA | Panel |
|-----|-----|-------|-----|-------|
| OAuth Server Admin | `admin` | `/admin/login` | Zorunlu | `/admin/*` |
| Resource Owner | `user` | OAuth authorize flow | Hayır | Yok (sadece consent) |

- **Administratorler:** `/admin/administrators` — server yöneticileri
- **Resource users:** `/admin/users` — OAuth ile oturum açan son kullanıcılar

## Yetkilendirme (iki katman)

### 1) OAuth Scopes (dış — OIDC)

`openid`, `profile`, `email` → consent ekranında gösterilir, userinfo/id_token claim'leri.

### 2) Permissions (dahili — Spatie)

`permissions` tablosu: `name`, `resource`, `access_scope` (`own` | `all`).

Örnek: `users.view` + resource `users` + scope `own` → sadece kendi kaydı.

## Admin Auth Akışı

1. `/admin/login` → email + password
2. 2FA kurulu değilse → `/admin/2fa/setup` (QR, zorunlu)
3. `/admin/2fa/challenge` → TOTP kodu
4. Başarısızsa → e-posta OTP (Mailpit)
5. Recovery codes (8 adet, tek kullanımlık)
6. `session('2fa_verified')` → admin panel erişimi

## Composer Paketleri (izinli)

- `laravel/passport`
- `spatie/laravel-permission`
- `pragmarx/google2fa-laravel`

Yeni paket eklemeden önce bu listeyi güncelleyin ve gerekçe yazın.

## Route Özeti

**Admin (web + session + 2FA):**

- `GET/POST /admin/login`, `/admin/2fa/*`, `POST /admin/logout`
- `GET /admin`, `/admin/administrators`, `/admin/users`, `/admin/clients`, `/admin/scopes`, `/admin/roles`, `/admin/permissions`

**OAuth (Passport):**

- `/oauth/authorize`, `/oauth/token`, `/.well-known/openid-configuration`, `/oauth/jwks`, `/api/userinfo`

**API (Bearer token):**

- `GET/POST/PUT/DELETE /api/users` — permission + own/all

## Ortam Değişkenleri (.env)

- `DB_CONNECTION=pgsql`
- `MAIL_HOST=mailpit`, `MAIL_PORT=1025`
- `QUEUE_CONNECTION=database`
- `APP_URL=http://localhost:8080`

## Agent Notları

- Container adları sabit değil; `docker compose ps` ile kontrol et
- `docker compose` (v2) kullan, `docker-compose` değil
- Değişiklik yaparken sade yapıyı koru; yeni soyutlama katmanı ekleme
- Grep/arama için gereksiz dosyalar `.cursorignore` içinde — oraya bak
- Bu dosyayı proje yapısı değişince güncelle
