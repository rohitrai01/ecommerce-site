# GreatKart — Django E-Commerce Website

A full-featured e-commerce web application built with **Django**. It supports product browsing by category with pagination and search, product variations (color/size), a shopping cart, user accounts with email verification, product reviews & ratings, order placement, and payment recording — with the real Django admin hidden behind a honeypot decoy login.

## Features

### Categories
- **`Category` model** — name, unique slug, description, and an optional category image
- **Site-wide navigation** — a context processor (`category.context_processors.menu_links`, wired into every template via `settings.py`) exposes all categories as `links`, so the category menu can appear on every page without each view passing it explicitly
- **Admin** — slug auto-populated from the category name

### Storefront
- **Home page** — lists available products
- **Category browsing** — products filtered by category, paginated (3 per page site-wide, 1 per page within a category)
- **Search** — keyword search across product name and description
- **Product detail** — shows the product, its image gallery, variations (color/size), and approved reviews; also flags whether the item is already in the visitor's cart and, for logged-in users, whether they've already purchased it
- **Reviews & ratings** — logged-in users can submit a review (subject, text, rating); resubmitting updates their existing review instead of creating a duplicate; each product exposes an average rating and review count; reviews store the submitter's IP and a `status` flag for moderation

### Accounts
- **Custom user model** — `accounts.Account` (email as the login field) replaces Django's default `User`
- **Registration & email activation** — sign-up sends a tokenized activation link via email
- **Login / logout**, **password reset via email**, **change password**
- **Profile** — address, city/state/country, and a profile picture; editable from the dashboard
- **Dashboard & order history** — order count, past orders, and a per-order breakdown with subtotal

### Cart & checkout
- **Guest and account carts** — guests are tracked by session key; logged-in users by account. Adding the same product+variation combination increments quantity instead of duplicating the line
- **Cart merge on login** — a guest's session cart is merged into the account's cart when they log in
- **Quantity controls** — increment via re-adding, decrement/remove via dedicated routes
- **Checkout** — cart subtotal, a flat 2% tax, and grand total (login required)

### Orders & payments
- **Place order** — from checkout, billing details (name, phone, email, address, city/state/country, order note) are captured via `OrderForm` and saved as an `Order` with `is_ordered=False`; an order number is generated as `YYYYMMDD` + the order's DB id
- **Payment capture** — a client-side payment flow (e.g. a JS payment button) posts the transaction ID, payment method, and status to `/orders/payments/`; this creates a `Payment` record, marks the `Order` as `is_ordered=True`, and links the two
- **Cart → order conversion** — on successful payment, each cart item is copied into `OrderProduct` (product, quantity, price, variations), product `stock` is decremented accordingly, and the user's cart is cleared
- **Order confirmation email** — sent to the customer after payment
- **Order complete page** — looks up the completed order and payment by order number/transaction id and shows a receipt with subtotal
- **Admin** — orders list with status/search filters, and an inline (read-only) breakdown of each order's line items; `Order.status` tracks New / Accepted / Completed / Cancelled independently of `is_ordered`

### Admin & security
- **Custom admin registrations** — product list view with inline product-gallery thumbnails, variation management with inline activation toggling
- **Hidden real admin** — `django-admin-honeypot` serves a decoy at `/admin/`; the real Django admin is mounted at `/securelogin/`
- **Session timeout** — sessions expire after 1 hour of inactivity and redirect to login (`django-session-timeout`)

### Infrastructure
- **Static & media storage** — both static and uploaded media files are served from **Amazon S3** (`django-storages` + `boto3`), not local disk
- **Email** — SMTP-based email backend for activation/reset emails
- **Deployment target** — configured for **AWS Elastic Beanstalk** (see `ALLOWED_HOSTS`), with database settings that switch to **RDS PostgreSQL** automatically when RDS environment variables are present

## Tech Stack

- **Backend:** Django 3.1 (Python)
- **Database:** SQLite by default; PostgreSQL via Amazon RDS in the Elastic Beanstalk environment (auto-detected from `RDS_*` env vars — see below)
- **Static/media storage:** Amazon S3 (`django-storages`, `boto3`) — **required**, not optional, even for local development (see note below)
- **Image handling:** Pillow
- **Config management:** `python-decouple` (`.env` file)
- **Deployment:** AWS Elastic Beanstalk

See `requirements.txt` for the full dependency list.

## Project Structure

```
greatkart/                # Project package
  ├── settings.py            # Django settings (S3, RDS/SQLite, email, sessions, custom user model)
  ├── urls.py                 # Root URLconf: admin honeypot, home, store, cart, accounts, orders
  ├── media_storages.py       # Custom S3 storage backend for media uploads
  ├── wsgi.py / asgi.py
  └── views.py                 # home view

accounts/                 # Custom Account/UserProfile models, auth, dashboard, order history
category/                 # Category model, admin, site-wide nav context processor
store/                    # Products, variations, gallery, reviews/ratings, storefront + search views
carts/                    # Shopping cart & cart items (session- and user-based)
orders/                   # Order/Payment/OrderProduct models, checkout → payment → confirmation flow

manage.py                 # Django management entry point
requirements.txt          # Python dependencies
.env-sample                # Sample environment variable file
data.json                  # Sample data fixture (categories, products, reviews, etc.)
```

## Getting Started

### Prerequisites

- Python 3.7+
- An **AWS S3 bucket** — the app hard-requires `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_STORAGE_BUCKET_NAME` (no defaults in `settings.py`), so it **will not start** without valid S3 credentials, even for local development
- An SMTP email account (e.g. Gmail app password) for sending activation/reset emails
- PostgreSQL, only if you want to run against RDS instead of the SQLite fallback

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

   Copy the sample env file:
   ```bash
   cp .env-sample .env
   ```

   `.env-sample` only lists the email settings — `settings.py` also reads AWS credentials directly from the environment (not listed in the sample file), so add those too:
   ```
   SECRET_KEY=your-django-secret-key
   DEBUG=True

   EMAIL_HOST=smtp.gmail.com
   EMAIL_PORT=587
   EMAIL_HOST_USER=your-email@example.com
   EMAIL_HOST_PASSWORD=your-email-app-password
   EMAIL_USE_TLS=True

   AWS_ACCESS_KEY_ID=your-aws-access-key
   AWS_SECRET_ACCESS_KEY=your-aws-secret-key
   AWS_STORAGE_BUCKET_NAME=your-s3-bucket-name
   ```

   > **Database:** by default the app uses local SQLite. To use PostgreSQL/RDS instead, set `RDS_DB_NAME`, `RDS_USERNAME`, `RDS_PASSWORD`, `RDS_HOSTNAME`, and `RDS_PORT` in the environment — `settings.py` switches automatically when these are present.

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

## Key URLs

| Path | Purpose |
|---|---|
| `/` | Home page |
| `/store/` | All available products (paginated) |
| `/store/category/<category_slug>/` | Products in a category |
| `/store/category/<category_slug>/<product_slug>/` | Product detail |
| `/store/search/?keyword=...` | Keyword search |
| `/store/submit_review/<product_id>/` | Submit or update a review (login required in practice) |
| `/cart/` | View cart |
| `/cart/add_cart/<product_id>/` | Add a product (with optional variation) to the cart |
| `/cart/remove_cart/<product_id>/<cart_item_id>/` | Decrease quantity by 1 |
| `/cart/remove_cart_item/<product_id>/<cart_item_id>/` | Remove the item entirely |
| `/cart/checkout/` | Checkout (login required) |
| `/accounts/register/`, `/accounts/login/`, `/accounts/logout/` | Auth |
| `/accounts/dashboard/` | User dashboard |
| `/accounts/activate/<uidb64>/<token>/` | Activate account from verification email |
| `/accounts/forgotPassword/`, `/accounts/resetPassword/` | Password reset flow |
| `/accounts/my_orders/`, `/accounts/order_detail/<order_id>/` | Order history |
| `/accounts/edit_profile/`, `/accounts/change_password/` | Account/profile management |
| `/orders/place_order/` | Capture billing details and generate the order number |
| `/orders/payments/` | Record a completed payment and convert the cart into order items |
| `/orders/order_complete/?order_number=...&payment_id=...` | Order receipt / confirmation page |

## Admin Access

- `/admin/` — a **decoy** login page (`django-admin-honeypot`)
- `/securelogin/` — the real Django admin
