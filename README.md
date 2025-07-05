# GDP Laravel Start – Template base per progetti Laravel gestionali

Questo repository rappresenta il punto di partenza per progetti Laravel sviluppati da **Generazione Digitale**.  
È configurato per:

-   Laravel 11 + Breeze (autenticazione base)
-   Bootstrap 5 + FontAwesome (via Vite)
-   Vite per asset e SCSS
-   Spatie Laravel Permissions per la gestione dei ruoli
-   Routing separato per area autenticata (`/admin`)

---

## 📁 Architettura del progetto

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Auth/                 ← Controller Breeze
│   │   └── Admin/               ← Controller loggati/protetti
│
resources/
└── views/
    ├── admin/                   ← Viste per utenti loggati
    │   └── dashboard.blade.php
    └── auth/                    ← Viste Breeze (login, register, ecc.)
```

---

## 🧩 Aggiungere una nuova entità (es. `Example`)

### 1. Creare il Model, Migration, Seeder e Factory

```bash
php artisan make:model Example -a
```

### 2. Spostare il controller generato

```bash
php artisan make:controller Admin/ExampleController -r --model=Example
```

> Puoi eliminare il controller `ExampleController` creato precedentemente nella root.

---

### 3. Definire le rotte protette in `routes/web.php`

All’interno del gruppo `/admin` (protetto da `auth` e `verified`):

```php
use App\Http\Controllers\Admin\ExampleController;

Route::resource('examples', ExampleController::class);
```

> Assicurati che l'import sia presente in cima al file.

---

### 4. Creare la migration e modificarla con i campi corretti

Poi eseguire:

```bash
php artisan migrate
```

---

### 5. Popolare lo seeder (opzionale)

Modifica `ExampleSeeder.php` come segue:

```php
use Faker\Generator as Faker;

public function run(Faker $faker): void {
    // esempio:
    \App\Models\Example::factory()->count(10)->create();
}
```

E lancialo:

```bash
php artisan db:seed --class=ExampleSeeder
```

---

### 6. Creare la vista

Crea una cartella `resources/views/admin/examples/`  
Al suo interno aggiungi ad esempio: `index.blade.php`

---

### 7. Modificare il controller

Nel metodo `index()` scrivi:

```php
public function index()
{
    $examples = Example::all();
    return view('admin.examples.index', compact('examples'));
}
```

---

## 🔐 Gestione Ruoli con Spatie

### Installazione (se non già fatto)

```bash
composer require spatie/laravel-permission
php artisan vendor:publish --tag="permission-config"
php artisan vendor:publish --tag="permission-migrations"
php artisan migrate
```

Aggiungi `HasRoles` al model User:

```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
    // ...
}
```

Ora puoi usare:

```php
$user->assignRole('admin');
$user->hasRole('staff');
```

---

## ⚙️ Note di configurazione

-   App name: viene preso da `.env` → `APP_NAME`
-   Favicon: da `public/favicon.ico`
-   Logo navbar: da `public/img/nav-logo.png`
-   Tutte le pagine dopo il login sono sotto `admin/`

---

## 📦 Comandi utili

```bash
# Primo avvio
composer install
npm install
cp .env.example .env
php artisan key:generate

# Build e avvio vite
npm run dev

# Avvio server
php artisan serve
```

# 🌐 Deployment Instructions – GDP Template

## ✅ Configure Vite Base Path

If your Laravel app is served from a subdirectory (e.g. `example.com/myapp/`), you must configure Vite to correctly resolve assets:

1. Open your `.env` file
2. Add or update the `VITE_APP_BASE` variable:

```env
VITE_APP_BASE=/myapp/
```

3. Ensure your `vite.config.js` contains:

```js
base: process.env.VITE_APP_BASE || '/',
```

> 📦 `VITE_APP_BASE` is automatically read by Vite when prefixed with `VITE_`.

---

## 📦 SSH + VPS Deployment Setup

### 1. SSH into your server

```bash
ssh root@your-server-ip
```

> Replace `your-server-ip` with the actual VPS IP address (e.g. `91.123.45.67`).

---

### 2. Create folder inside `/var/www/`

```bash
cd /var/www/
mkdir -p generazionedigitaleprogrammi/myapp
```

Copy or deploy your Laravel app inside that path (e.g. via `git clone` or `scp`).

---

### 3. NGINX Configuration

Create a new NGINX config file inside:

```bash
/etc/nginx/sites-available/
```

Example: `/etc/nginx/sites-available/myapp.com`

Paste this example config:

```nginx
server {
    listen 443 ssl;
    server_name myapp.generazionedigitaleprogrammi.com;

    root /var/www/generazionedigitaleprogrammi/myapp/public;
    index index.php index.html;

    ssl_certificate /etc/letsencrypt/live/myapp.generazionedigitaleprogrammi.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.generazionedigitaleprogrammi.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\. {
        deny all;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log /var/log/nginx/myapp_access.log;
    error_log  /var/log/nginx/myapp_error.log;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name myapp.generazionedigitaleprogrammi.com;
    return 301 https://$host$request_uri;
}
```

---

### 4. Enable the NGINX site

```bash
ln -s /etc/nginx/sites-available/myapp.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

---

### 5. Generate SSL (only once per domain)

Install Certbot if not yet installed:

```bash
apt install certbot python3-certbot-nginx
```

Run Certbot:

```bash
certbot --nginx -d myapp.generazionedigitaleprogrammi.com
```

---

## 🛠 Laravel Post-Deployment Setup

After uploading the Laravel project:

```bash
cd /var/www/generazionedigitaleprogrammi/myapp
cp .env.example .env
nano .env   # <- set DB config, VITE_APP_BASE, APP_URL, etc.
php artisan key:generate
composer install
npm install && npm run build
php artisan migrate --seed
php artisan config:cache
php artisan route:cache
```

---

## 📄 Notes

-   Ensure `public/` is the root in NGINX, **not** the Laravel base path.
-   Make sure `APP_URL` and `VITE_APP_BASE` match the actual served subpath.
-   Laravel logs will go in `storage/logs/`
-   You may need to `chown -R www-data:www-data .` after deploy.
