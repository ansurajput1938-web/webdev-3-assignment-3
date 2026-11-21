# Inventory and Data Management System

**Web Dev III — Assignment 3 (Unit 3)**  
Backend REST API using Node.js, Express.js, MongoDB, Mongoose and dotenv. No frontend or admin panel.

## Requirements
- Node.js 18+ and npm
- MongoDB Community Server running locally, or a MongoDB Atlas connection string

## Setup
1. Extract/open this project folder in VS Code.
2. Open the VS Code terminal in this folder.
3. Install dependencies:
   ```bash
   npm install
   ```
4. Create your environment file:
   ```bash
   cp .env.example .env
   ```
   On Windows, copy `.env.example` to `.env` manually.
5. Edit `.env` and set `MONGODB_URI`. Local default:
   `mongodb://127.0.0.1:27017/inventory_management`
6. Start MongoDB, then start the API:
   ```bash
   npm start
   ```
7. Open `http://localhost:5000/`. The API base URL is `http://localhost:5000/api/products`.

## Seed sample products
In a second terminal, run:
```bash
npm run seed
```
This inserts/updates three sample records from `sample-data/products.json` by SKU.

## Product fields and validation
| Field | Rules |
|---|---|
| name | Required string, 2–100 chars |
| sku | Required, unique, uppercase, 2–40 chars, letters/numbers/hyphens |
| category | Required string, 2–60 chars |
| price | Required finite number, minimum 0 |
| quantity | Required integer, minimum 0 |
| reorderLevel | Required integer, minimum 0 |
| supplier | Required string, 2–100 chars |

A unique MongoDB index prevents duplicate SKUs. Stock updates use an atomic conditional update, so stock cannot drop below zero.

## API endpoints
All endpoints use JSON. For POST/PUT/PATCH, select **Body → raw → JSON** in Postman.

### 1. Create product
`POST /api/products` — returns **201 Created**.
```json
{
  "name": "Gel Pen Pack of 15",
  "sku": "STNY-PN-713",
  "category": "Stationery",
  "price": 120,
  "quantity": 40,
  "reorderLevel": 20,
  "supplier": "Cello Pens"
}
```
Example response:
```json
{
  "success": true,
  "message": "Product created",
  "product": {
    "_id": "GENERATED_MONGODB_ID",
    "name": "Gel Pen Pack of 15",
    "sku": "STNY-PN-713",
    "category": "Stationery",
    "price": 120,
    "quantity": 40,
    "reorderLevel": 20,
    "supplier": "Cello Pens"
  }
}
```

### 2. Get all products
`GET /api/products` — returns **200 OK** with `total`, `page`, `limit`, `totalPages`, `count`, `products`.

### 3. Filtering, sorting and pagination
`GET /api/products?category=Stationery&minPrice=50&maxPrice=500&search=pen&page=1&limit=5&sortBy=price&order=asc`
- Filters: `category`, `minPrice`, `maxPrice`, `lowStock=true`, `search` (name/SKU/supplier)
- Pagination: `page` (default 1), `limit` (default 10, maximum 100)
- Sorting: `sortBy=name|sku|category|price|quantity|createdAt`, `order=asc|desc`

### 4. Get one product
`GET /api/products/:id` — returns **200 OK**, or **400** for invalid ID / **404** if not found.

### 5. Update product
`PUT /api/products/:id` — send any subset of product fields; validators run. Returns **200 OK**.
```json
{
  "price": 140,
  "quantity": 60
}
```

### 6. Delete product
`DELETE /api/products/:id` — returns **200 OK** with deletion message and ID.

### 7. Low-stock report
`GET /api/products/low-stock` — returns **200 OK** and products where `quantity <= reorderLevel`.
```json
{
  "success": true,
  "count": 1,
  "lowStockItems": []
}
```

### 8. Category-wise inventory summary
`GET /api/products/summary` — returns **200 OK**. Uses MongoDB aggregation `$group`, `$sum`, `$avg`, `$multiply`, `$round` and `$project`.
```json
{
  "success": true,
  "categories": 2,
  "summary": [
    {
      "category": "Electronics",
      "totalItems": 1,
      "totalQuantity": 25,
      "totalStockValue": 14975,
      "avgPrice": 599
    }
  ]
}
```

### 9. Adjust stock
`PATCH /api/products/:id/stock` — positive `change` restocks; negative `change` records a sale. Returns **200 OK**.
Restock:
```json
{ "change": 10 }
```
Sale:
```json
{ "change": -3 }
```
A zero/non-integer change returns **400**. An adjustment that would make quantity negative returns **400**. Missing product returns **404**.

## Postman practical demonstration
1. Create a collection named **Inventory Management API**.
2. Set collection variable `baseUrl` to `http://localhost:5000`.
3. Create requests using the methods and paths above, prefixing each path with `{{baseUrl}}`.
4. Run `POST /api/products` with the sample body. Copy the returned `product._id`.
5. Use that ID in `GET`, `PUT`, `DELETE`, and `PATCH` requests.
6. Run `GET /api/products?category=Stationery&page=1&limit=5&sortBy=price&order=asc`.
7. Run `GET /api/products/low-stock` and `GET /api/products/summary`.
8. Demonstrate duplicate SKU (expected **409 Conflict**), invalid/negative values (expected **400 Bad Request**) and a sale larger than available stock (expected **400 Bad Request**).

## HTTP status codes
- `200` successful read/update/delete/stock adjustment
- `201` product created
- `400` invalid input, invalid ID, validation failure, insufficient stock
- `404` route or product not found
- `409` duplicate SKU
- `500` unexpected server error

## Project structure
```text
inventory-data-management-system/
├── server.js
├── package.json
├── .env.example
├── .gitignore
├── README.md
├── sample-data/
│   └── products.json
├── scripts/
│   └── seed.js
├── test/
│   └── schema.test.js
└── src/
    ├── app.js
    ├── config/db.js
    ├── controllers/productController.js
    ├── middleware/asyncHandler.js
    ├── middleware/errorMiddleware.js
    ├── models/Product.js
    └── routes/productRoutes.js
```

## Tests
Run schema validation tests (no database needed):
```bash
npm test
```
The API integration demonstration requires a running MongoDB instance and a valid `MONGODB_URI`.


Name- Deepanshu 
Roll Number- 2501010227
Section- B.Tech CSE Core A