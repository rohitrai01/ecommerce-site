# GreatKart — Django E-Commerce Website

A full-featured e-commerce web application built with **Django**. It supports product browsing by category, product variations (size/color), a shopping cart, user accounts with email verification, order placement, and payment recording — with an admin panel protected by a honeypot login page.

## Features

- **Accounts** — user registration, login, profile management, email-based account activation/password reset
- **Category & Store** — product categories, product listings, product image gallery, product variations (e.g. size/color), customer reviews & ratings
- **Cart** — add/update/remove items, per-session and per-user cart handling
- **Orders** — checkout flow, order and order-item records, payment tracking
- **Admin security** — `django-admin-honeypot` to disguise/protect the real admin login
- **Media storage** — Amazon S3 support via `django-storages` and `boto3`
- **Email** — SMTP email support (order confirmations, account verification) via Django's email backend
- **Session security** — auto session expiry via `django-session-timeout`

## Tech Stack

- **Backend:** Django (Python)
- **Database:** PostgreSQL (`psycopg2-binary`)
- **Media/Static storage:** AWS S3 (`django-storages`, `boto3`)
- **Image handling:** Pillow
- **Config management:** `python-decouple` (`.env` file)

See `requirements.txt` for the full dependency list.

## Project Structure (key apps)

```
greatkart/           # Project settings module
accounts/            # User accounts, profiles, authentication
category/            # Product categories
store/                # Products, product gallery, variations, reviews/ratings
carts/                # Shopping cart & cart items
orders/               # Orders, order products, payments
manage.py             # Django management entry point
requirements.txt       # Python dependencies
.env-sample            # Sample environment variable file
data.json              # Sample data fixture (categories, products, etc.)
```

## Getting Started

### Prerequisites

- Python 3.7+
- PostgreSQL
- An AWS S3 bucket (if using S3 for media/static storage)
- An SMTP email account (e.g. Gmail app password) for sending emails

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/rohitrai01/ecommerce-site.git
   cd ecommerce-site
   ```

2. **Create and activate a virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate      # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables**

   Copy the sample env file and fill in your own values:
   ```bash
   cp .env-sample .env
   ```

   Update `.env` with:
   ```
   SECRET_KEY=your-django-secret-key
   DEBUG=True
   EMAIL_HOST=smtp.gmail.com
   EMAIL_PORT=587
   EMAIL_HOST_USER=your-email@example.com
   EMAIL_HOST_PASSWORD=your-email-app-password
   EMAIL_USE_TLS=True
   ```

   > Note: if your `settings.py` also expects database or AWS S3 credentials (e.g. `DATABASE_URL`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_STORAGE_BUCKET_NAME`), add those to `.env` as well — check `greatkart/settings.py` for the exact variable names it reads.

5. **Set up the database**
   ```bash
   python manage.py migrate
   ```

6. **(Optional) Load sample data**

   The repo includes a `data.json` fixture with sample categories, products, and reviews:
   ```bash
   python manage.py loaddata data.json
   ```

7. **Create a superuser**
   ```bash
   python manage.py createsuperuser
   ```

8. **Run the development server**
   ```bash
   python manage.py runserver
   ```

   Visit `http://127.0.0.1:8000/` to view the site.

## Admin Access

Since `django-admin-honeypot` is installed, the default `/admin/` URL serves a decoy login page. Check `greatkart/urls.py` for the actual admin path configured for this project.




