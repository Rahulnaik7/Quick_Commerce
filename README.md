# Quick Commerce Delivery System

A full-stack quick-commerce web app: customers browse a grocery catalog, order in a few clicks, and follow their delivery on a live map, while admins manage products, orders and delivery staff.

Built with **React**, **Node.js/Express** and **PostgreSQL** as a course project for CS 317 and CS 349 at IIT Bombay.

---

## Features

**Customer**
- Sign up and log in with passwords hashed using bcrypt and sessions held in a JWT stored in an httpOnly cookie
- Browse products by category, see featured items, and search the catalog
- Add, update and remove cart items, then check out with card, UPI or cash on delivery
- Save multiple delivery addresses (with latitude and longitude), and edit or delete them
- Rate and review products, and mark other reviews as helpful
- View past orders and track an active order

**Order tracking**
- Status timeline for every order (placed, out for delivery, delivered and so on)
- Map view built with Leaflet showing the store, the delivery partner and the customer's address, with the route between them
- The tracking page refreshes the delivery partner's location every 30 seconds
- An estimated delivery time is set when the order is placed and updated from the partner's distance to the customer

**Admin panel** (restricted to admin accounts)
- Add, edit and delete products
- View all orders and update their status
- Add and remove delivery personnel
- View users and grant or revoke admin rights

---

## Tech stack

| Layer    | Technology |
|----------|------------|
| Frontend | React 18, React Router 6, Leaflet, Create React App |
| Backend  | Node.js, Express 4, `pg` (node-postgres), bcrypt, jsonwebtoken, cookie-parser, CORS, dotenv |
| Database | PostgreSQL |

## Architecture

```
React app (localhost:3000)
        │  REST calls with a JWT cookie
        ▼
Express API (localhost:5000/api)
   ├── authenticateToken middleware  →  any logged-in user
   ├── isAdmin middleware            →  /api/admin/* routes
        │  SQL through a pg connection pool
        ▼
PostgreSQL (quick_commerce database)
```

In production (`NODE_ENV=production`), Express also serves the built React app from `frontend/build`.

## Project structure

```
Quick_Commerce/
├── backend/
│   ├── app.js          # Express server: every API route and middleware
│   ├── DDL.sql         # Database schema
│   ├── data.sql        # Sample data (users, categories, products, stores)
│   └── package.json
└── frontend/
    ├── public/
    └── src/
        ├── pages/      # Home, Products, Cart, Orders, DeliveryTracking, AdminPanel, ...
        ├── components/ # Navbar, ProductCard, AddressForm, ReviewForm, ...
        ├── contexts/   # Theme context
        ├── config/     # API base URL
        └── css/
```

---

## Getting started

### Prerequisites
- Node.js and npm
- PostgreSQL

### 1. Set up the database

```bash
createdb quick_commerce
psql -d quick_commerce -f backend/DDL.sql
psql -d quick_commerce -f backend/data.sql
```

`data.sql` includes sample products and users, including an admin account (`admin@quickcommerce.com`).

### 2. Run the backend

Copy `backend/.env.example` to `backend/.env` and fill in your values. The server will not start without `DB_USER`, `DB_PASSWORD` and `JWT_SECRET`.

```env
DB_USER=your_postgres_user
DB_PASSWORD=your_postgres_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=quick_commerce
JWT_SECRET=replace-with-a-long-random-string
PORT=5000
NODE_ENV=development
```

Then:

```bash
cd backend
npm install
npm run dev        # starts with nodemon on http://localhost:5000
```

### 3. Run the frontend

```bash
cd frontend
npm install
npm start          # opens http://localhost:3000
```

The frontend calls `http://localhost:5000/api` by default. To use a different backend, set `REACT_APP_API_URL` in `frontend/.env`.

---

## API overview

All routes are under `/api`. 🔒 means a login is required, and 👑 means an admin account is required.

| Area | Endpoints |
|------|-----------|
| Auth | `POST /register`, `POST /login`, `POST /logout`, `GET /isLoggedIn` |
| Profile and addresses | 🔒 `GET /profile`, `POST /addresses`, `PUT /addresses/:addressId`, `DELETE /addresses/:addressId` |
| Catalog | `GET /products`, `GET /products/featured`, `GET /products/search`, `GET /products/:productId`, `GET /categories`, `GET /products/category/:categoryId` |
| Cart | 🔒 `GET /cart`, `POST /cart/items`, `PUT /cart/items/:itemId`, `DELETE /cart/items/:itemId` |
| Reviews | `GET /products/:productId/reviews`, 🔒 `POST /products/:productId/reviews`, `POST /reviews/:reviewId/helpful` |
| Orders and tracking | 🔒 `POST /orders`, `GET /orders`, `GET /orders/:orderId/tracking`, `POST /orders/:orderId/location`, `GET /store-location` |
| Admin | 👑 `GET /admin/users`, `PUT /admin/users/:userId/admin`, `GET/POST /admin/products`, `PUT/DELETE /admin/products/:productId`, `GET /admin/orders`, `PUT /admin/orders/:orderId/status`, `GET/POST /admin/delivery-personnel`, `DELETE /admin/delivery-personnel/:id` |

## Database schema

Tables in `backend/DDL.sql`:

- **Users and addresses:** `users`, `addresses`
- **Catalog:** `categories`, `products`, `reviews`, `review_helpful`
- **Shopping:** `cart`, `cart_items`, `orders`, `order_items`
- **Delivery:** `delivery_personnel`, `order_tracking`, `delivery_locations`, `store_locations`, `stores`

---

## Possible improvements

- Push location updates over WebSockets instead of polling every 30 seconds
- Calculate ETA from road distance rather than straight-line distance
- Split `app.js` into route, controller and data-access modules
- Add automated tests for the API

## Credits

Developed as a team course project at IIT Bombay. The original repository is [SanskarSharma123/Quick_Commerce](https://github.com/SanskarSharma123/Quick_Commerce).
