# GDP Laravel Start – Template base per progetti Laravel gestionali

Questo repository rappresenta il punto di partenza per progetti Laravel sviluppati da **Generazione Digitale**.  
È configurato per:

- Laravel 11 + Breeze (autenticazione base)
- Bootstrap 5 + FontAwesome (via Vite)
- Vite per asset e SCSS
- Spatie Laravel Permissions per la gestione dei ruoli
- Routing separato per area autenticata (`/admin`)

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

- App name: viene preso da `.env` → `APP_NAME`
- Favicon: da `public/favicon.ico`
- Logo navbar: da `public/img/nav-logo.png`
- Tutte le pagine dopo il login sono sotto `admin/`

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
