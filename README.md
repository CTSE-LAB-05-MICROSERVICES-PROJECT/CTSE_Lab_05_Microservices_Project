# Microservices Project — CTSE Lab 05

### Student ID - IT22643568
### Student Name - ABEYKOON A.M.D.G.A.S

A three-service microservices architecture with an API Gateway and separate database for each service, all containerised with Docker.

---

## Tech Stack

| Service | Language / Runtime | Framework | ORM / DB Driver | Port |
|---|---|---|---|---|
| **Item Service** | Python 3.13 | FastAPI | SQLAlchemy (asyncpg) | 8081 |
| **Order Service** | Go 1.25 | Gin | GORM + pgx | 8082 |
| **Payment Service** | Java 21 | Spring Boot 3 | Spring Data JPA (Hibernate) | 8083 |
| **API Gateway** | Node.js | Express Gateway | — | 8080 |
| **Database** | — | PostgreSQL 18 | — | 5432 |

All three services share a single PostgreSQL instance with separate databases (`item_service_db`, `order_service_db`, `payment_service_db`).

---

## Repository Structure & Submodules

Each service lives in its own Git repository and is included here as a **Git submodule** under `services/`:

```
services/
  item-service/     ← submodule
  order-service/    ← submodule
  payment-service/  ← submodule
```

---

## Getting Started

### 1. Clone with Submodules

```bash
git clone --recurse-submodules <repo-url>
cd Microservices_Project_CTSE_Lab_05
```

If you already cloned without `--recurse-submodules`, initialise the submodules manually:

```bash
git submodule update --init --recursive
```

### 2. Create Environment Files

**`services/item-service/.env`**
```env
PORT=8081
DB_URL=postgresql+asyncpg://admin:postgres@postgres:5432/item_service_db
```

**`services/order-service/.env`**
```env
PORT=:8082
DB_URL=host=postgres user=admin password=postgres dbname=order_service_db port=5432 sslmode=disable
```

The Payment Service reads its database configuration directly from `application.properties` — no `.env` file is required.

### 3. Start All Services

```bash
docker compose up --build
```

This brings up PostgreSQL, all three services, and the API Gateway. The `postgres` container uses an init script (`configs/init/databases.sh`) to create all three databases automatically.

Once running, all services are accessible **directly** on their own ports and also through the API Gateway on port **8080**.

---

## API Endpoints & curl Examples

All examples below use the **API Gateway** at `http://localhost:8080`. You can also hit the services directly by substituting the port (8081 / 8082 / 8083).

---

### Item Service

**Get all items**
```bash
curl -X GET http://localhost:8080/items/
```
Expected response:
```json
[]
```

**Create an item**
```bash
curl -X POST http://localhost:8080/items/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Laptop", "price": 1299.99, "customer_id": "cust-001"}'
```
Expected response:
```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Laptop",
  "price": 1299.99,
  "customer_id": "cust-001",
  "created_at": "2026-03-03T10:00:00",
  "updated_at": "2026-03-03T10:00:00"
}
```

**Get an item by ID**
```bash
curl -X GET http://localhost:8080/items/a1b2c3d4-e5f6-7890-abcd-ef1234567890
```
Expected response:
```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Laptop",
  "price": 1299.99,
  "customer_id": "cust-001",
  "created_at": "2026-03-03T10:00:00",
  "updated_at": "2026-03-03T10:00:00"
}
```

---

### Order Service

**Get all orders**
```bash
curl -X GET http://localhost:8080/orders/
```
Expected response:
```json
[]
```

**Create an order**
```bash
curl -X POST http://localhost:8080/orders/ \
  -H "Content-Type: application/json" \
  -d '{"item": "Laptop", "quantity": 2, "customerID": "cust-001"}'
```
Expected response:
```json
{
  "id": "f1e2d3c4-b5a6-7890-abcd-123456789abc",
  "item": "Laptop",
  "quantity": 2,
  "customerID": "cust-001",
  "createdAt": "2026-03-03T10:05:00Z",
  "updatedAt": "2026-03-03T10:05:00Z"
}
```

**Get an order by ID**
```bash
curl -X GET http://localhost:8080/orders/f1e2d3c4-b5a6-7890-abcd-123456789abc
```
Expected response:
```json
{
  "id": "f1e2d3c4-b5a6-7890-abcd-123456789abc",
  "item": "Laptop",
  "quantity": 2,
  "customerID": "cust-001",
  "createdAt": "2026-03-03T10:05:00Z",
  "updatedAt": "2026-03-03T10:05:00Z"
}
```

---

### Payment Service

**Get all payments**
```bash
curl -X GET http://localhost:8080/payments
```
Expected response:
```json
[]
```

**Process a payment**
```bash
curl -X POST http://localhost:8080/payments/process \
  -H "Content-Type: application/json" \
  -d '{"orderId": 1, "amount": 2599.98, "method": "CREDIT_CARD"}'
```
Expected response (`201 Created`):
```json
{
  "id": 1,
  "orderId": 1,
  "amount": 2599.98,
  "method": "CREDIT_CARD"
}
```

**Get a payment by ID**
```bash
curl -X GET http://localhost:8080/payments/1
```
Expected response:
```json
{
  "id": 1,
  "orderId": 1,
  "amount": 2599.98,
  "method": "CREDIT_CARD"
}
```

---

## Stopping the Project

```bash
docker compose down
```

To also remove the persisted database volume:

```bash
docker compose down -v
```
