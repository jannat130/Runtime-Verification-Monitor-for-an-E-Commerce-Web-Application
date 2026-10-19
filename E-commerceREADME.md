#E-Commerce Web Application

A Laravel-based e-commerce storefront supporting product browsing, cart management, discount codes, and checkout with Stripe payment. Built as the target application for a Runtime Verification monitoring project.

## Tech Stack

- **Backend:** PHP / Laravel
- **Database:** MySQL (via phpMyAdmin / XAMPP)
- **Frontend:** Blade templates, jQuery
- **Payment:** Stripe
- **Local server:** XAMPP (Apache + MySQL)

## Prerequisites

Install these before starting, if you don't have them already:

- [XAMPP](https://www.apachefriends.org) (includes Apache, MySQL, and phpMyAdmin)
- [Composer](https://getcomposer.org) (PHP dependency manager)
- PHP 7.4 or 8.x (bundled with XAMPP)

## Setup Instructions

### 1. Start XAMPP

Open the XAMPP Control Panel and start the **Apache** and **MySQL** modules.

### 2. Place the project in `htdocs`

Copy the entire project folder into your XAMPP `htdocs` directory, so the final path looks like:

```
C:\xampp\htdocs\ecommerce
```

> **Avoid special characters** (`&`, `%`, `$`, etc.) anywhere in the folder path — they can silently break scripts and command-line tools later. Stick to letters, numbers, underscores, and hyphens.

### 3. Import the database

The database export is provided in the `access` folder included with this project (a `.sql` file).

1. Open `http://localhost/phpmyadmin` in your browser.
2. Click **New** to create a database. Name it to match `DB_DATABASE` in the `.env` file (see Step 4).
3. Select the new database, go to the **Import** tab.
4. Click **Choose File**, select the `.sql` file from the `access` folder.
5. Click **Go** and wait for the import to complete.

### 4. Configure environment variables

1. In the project root, copy `.env.example` to a new file named `.env` (if `.env` doesn't already exist).
2. Open `.env` and set your database credentials to match what you created in Step 3:
   ```
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=databasename
   DB_USERNAME=root
   DB_PASSWORD=
   ```
   (Default XAMPP MySQL username is `root` with a blank password, unless you've changed it.)

   ```


Open a terminal, navigate to the project folder, and run:

```

### 7. Visit the site

Open your browser and go to:

```
http://localhost/ecommerce
```

You should see the storefront homepage. Try browsing a product and adding it to your cart to confirm the database connection is working.

## Key Features

- Product catalogue with categories, search, and sorting
- Shopping cart with live stock validation
- Discount/coupon codes at checkout
- Checkout with Stripe payment
- Order tracking and customer account pages
- Admin back-office for managing products, orders, purchases, and stock (existing LIMS-based admin panel)

## Notes on Checkout

The checkout page has been simplified to a single delivery option (free shipping) and a single payment method (Stripe), to keep the demo flow focused. These are set via hidden form fields rather than removed from the underlying logic, so no other part of the application needed to change.

## Related Component: Runtime Verification Monitor

This application is instrumented to write structured event logs (`storage/logs/events.log`) at key checkout points — order placement, stock checks, discount application, and payment recording. A separate Python-based monitor (provided alongside this project, in the `rv_monitor` folder) reads this log and checks it against a set of business rules in real time, displaying results on a live dashboard.

See the **RV Monitor Setup Guide** (included separately) for instructions on running that component. It is independent of this application and does not need to be running for the storefront itself to work.

## Troubleshooting

**Blank page or 500 error on load**
Run `php artisan config:clear` and `php artisan cache:clear`, then reload.

**"SQLSTATE" or database connection errors**
Double-check `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD` in `.env` match what you set up in phpMyAdmin, and that the MySQL module is running in the XAMPP Control Panel.

**Product images not showing**
Make sure Step 6 (`php artisan storage:link`) was run, and that the `public/storage` symlink exists.

**Changes to `.env` don't seem to take effect**
Run `php artisan config:clear` after every `.env` edit — Laravel caches configuration and won't pick up changes otherwise.
