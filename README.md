# Travel Tea — Distributor Management System

Private web portal for Travel Tea and its distributors. Laravel 12 · Livewire 4 · Tailwind CSS 4 · MySQL 8.
There is no public site and no sign-up: admins create every account.

## Setup

```bash
composer install
npm install && npm run build
cp .env.example .env && php artisan key:generate   # set DB_* in .env
php artisan migrate --seed                         # roles, permissions, units, default Super Admin
php artisan storage:link
php artisan db:seed --class=DemoSeeder             # optional: demo products, 3 distributors, orders, sales
```

Default Super Admin: `admin@traveltea.com` / `password` (set `ADMIN_EMAIL` / `ADMIN_PASSWORD` in `.env` before seeding, and change the password after the first login).
Demo distributor logins (DemoSeeder only): `ali@example.com`, `madina@example.com`, `multan@example.com` / `password`.

Run locally: `php artisan serve`, then open http://127.0.0.1:8000.

## Production

- Web root → `public/`, HTTPS, `APP_ENV=production`, `APP_DEBUG=false`.
- Cron: `* * * * * cd /path/to/travel-tea && php artisan schedule:run >> /dev/null 2>&1`
  (runs `app:backup` daily at 02:00 → `storage/app/backups`, keeps 30 days; also copy these off the server).
- Email (password reset): set `MAIL_*` in `.env`.
- Logo: official Travel Tea logo in `public/images` (logo.png, logo-full.png with Urdu tagline, logo-sm.png, logo-icon.png) plus `public/favicon.ico` and `apple-touch-icon.png`.

## How it fits together

Every business action is a service in `app/Services` running inside one database transaction:

| Action | Stock | Ledger |
|---|---|---|
| Supply order confirm | reserved in warehouse | distributor debited |
| Dispatch / Deliver | warehouse out / distributor in (at net cost) | — |
| Cancel order | reversed | credited back |
| POS sale | distributor out (weighted avg cost), shop in if tracked | customer debited, payment credited |
| Payment | — | credited (bounced cheque debits back) |
| Returns | back to stock (saleable or damaged) | credited |

- Stock is a ledger (`stock_movements`); `stock_balances` is a projection updated in the same transaction. Base unit is the packet; bundles and cartons are set per product.
- Balances are always computed from `ledger_entries`, never typed in.
- Distributor data isolation: `App\Models\Concerns\BelongsToDistributor` global scope and Livewire persistent middleware.
- Pricing rules: `App\Services\PricingService` (margin + trade offers on the gross price; best non-stackable offer; discount code last).
- Reports: `app/Reports`, registered in `ReportRegistry`; each exports to Excel, CSV, PDF and print.
- Permissions: `config/permissions.php` → `php artisan db:seed --class=PermissionSeeder` after adding new ones.

## Tests

`php artisan test` (uses the MySQL database `travel_tea_test`, see `phpunit.xml`).
