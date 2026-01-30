# Async Payment Gateway 

A **production-ready payment gateway system** built using asynchronous processing, distributed job queues, secure webhook delivery, an embeddable checkout SDK, and complete refund lifecycle management.

This project demonstrates how modern fintech platforms handle **non-blocking payments, background workers, webhook reliability, and system fault tolerance** at scale.

---

##  Core Capabilities

###  Asynchronous Payment Processing

* Payments are processed using **Bull job queues backed by Redis**
* API responds immediately without blocking
* Background workers finalize payment execution

**Queue separation:**

* Payment queue
* Webhook delivery queue
* Refund processing queue

**Payment state flow:**

```
pending → success | failed
```

---

###  Webhook Delivery Engine

* Cryptographically signed webhooks using **HMAC-SHA256**
* Signature included in:

```
X-Webhook-Signature
```

* Automatic retry mechanism with exponential backoff
* Maximum **5 delivery attempts**

**Retry schedule**

| Environment | Retry Intervals      |
| ----------- | -------------------- |
| Test        | 5s → 10s → 15s → 20s |
| Production  | 1m → 5m → 30m → 2h   |

**Supported events**

* `payment.success`
* `payment.failed`
* `refund.processed`

Merchants can configure webhook URL and secret directly from the dashboard.

---

###  Embeddable Checkout SDK

* JavaScript SDK for merchant websites
* Modal / iframe-based checkout
* No redirect away from merchant site
* Secure communication using `postMessage`
* Fully responsive UI

**SDK lifecycle callbacks**

* `onSuccess`
* `onFailure`
* `onClose`

---

###  Refund Management

* Supports both **full and partial refunds**
* Refunds processed asynchronously
* Validation prevents refund amounts exceeding payment total
* Webhook notification sent after completion

**Refund lifecycle**

```
pending → processed
```

---

###  Idempotency Protection

* Duplicate payment requests prevented using `Idempotency-Key`
* Cached response returned for repeated requests
* Keys expire automatically after **24 hours**
* Scoped per merchant to avoid cross-account conflicts

---

##  Architecture Overview

```
Async Payment Gateway
│
├── API Server (Express.js | Port 8000)
│   ├── REST APIs
│   ├── Authentication middleware
│   ├── Bull job producers
│   └── PostgreSQL persistence
│
├── Worker Services
│   ├── paymentWorker.js
│   ├── webhookWorker.js
│   └── refundWorker.js
│
├── Redis (Port 6379)
│   └── Job queue backend
│
├── Frontend
│   ├── Merchant Dashboard (Port 3000)
│   ├── Checkout Application (Port 3001)
│   └── Checkout SDK (checkout.js)
│
└── Test Utilities
    └── Local webhook receiver (Port 3002)
```

---



##  Getting Started

### Prerequisites

* Docker Desktop
* Git

---

### Clone Repository

```bash
git clone https://github.com/your-username/async-payment-gateway.git
cd async-payment-gateway
```

---

### Environment Setup

```bash
cp .env.example .env
```

Update values if required.

---

### Start Application

```bash
docker-compose up -d --build
```

Verify containers:

```bash
docker-compose ps
```

---

##  Service Endpoints

| Service   | URL                                            |
| --------- | ---------------------------------------------- |
| API       | [http://localhost:8000](http://localhost:8000) |
| Dashboard | [http://localhost:3000](http://localhost:3000) |
| Checkout  | [http://localhost:3001](http://localhost:3001) |
| Redis     | localhost:6379                                 |

---

### Health Check

```bash
curl http://localhost:8000/health
```

Expected response:

```json
{
  "status": "healthy"
}
```

---

##  Test Merchant Credentials

Seeded automatically during startup.

| Field      | Value                                       |
| ---------- | ------------------------------------------- |
| Email      | [test@example.com](mailto:test@example.com) |
| API Key    | key_test_abc123                             |
| API Secret | secret_test_xyz789                          |

---

##  API Overview

### Authentication Headers

```
X-Api-Key
X-Api-Secret
```

---

### Create Order

`POST /api/v1/orders`

```json
{
  "amount": 50000,
  "currency": "INR",
  "receipt": "receipt_001"
}
```

---

### Create Payment

`POST /api/v1/payments`

UPI example:

```json
{
  "order_id": "order_xxx",
  "method": "upi",
  "vpa": "user@paytm"
}
```

Card example:

```json
{
  "order_id": "order_xxx",
  "method": "card",
  "card": {
    "number": "4111111111111111",
    "expiry_month": "12",
    "expiry_year": "2025",
    "cvv": "123",
    "holder_name": "John Doe"
  }
}
```

Payments return immediately with status **pending** and complete asynchronously.

---

##  Async Processing Flow

```
Client request
 → Payment stored as pending
 → Job added to Redis queue
 → Worker processes payment
 → Status updated
 → Webhook queued
 → Merchant notified
```

---

##  Webhook Verification

Each webhook contains an HMAC signature.

Example verification:

```js
const crypto = require("crypto");

const expected = crypto
  .createHmac("sha256", WEBHOOK_SECRET)
  .update(JSON.stringify(req.body))
  .digest("hex");

if (signature !== expected) {
  return res.status(401).send("Invalid signature");
}
```

---

##  SDK Integration Example

```html
<script src="http://localhost:3001/checkout.js"></script>

<script>
const gateway = new PaymentGateway({
  key: "key_test_abc123",
  orderId: "order_123",
  onSuccess: (id) => console.log("Success:", id),
  onFailure: (err) => console.log(err),
  onClose: () => console.log("Closed")
});

gateway.open();
</script>
```

---

##  Test Mode

```env
TEST_MODE=true
TEST_PAYMENT_SUCCESS=true
TEST_PROCESSING_DELAY=1000
```

Provides predictable behavior for evaluation.

---

##  Database Tables

* merchants
* orders
* payments
* refunds
* webhook_logs
* idempotency_keys

---

##  Project Structure

```
async-payment-gateway/
├── backend/
├── frontend/dashboard/
├── checkout/
├── test-merchant/
├── docker-compose.yml
├── README.md
├── .env.example
└── submission.yml
```

---

##  Deliverable 2 Coverage

* Async job queues
* Redis-backed workers
* Webhook retries with HMAC security
* Refund processing pipeline
* Idempotency handling
* Checkout SDK
* Merchant dashboard
* Dockerized environment
* Automated evaluation readiness

---

##  Summary

This system mirrors how modern payment platforms operate internally:

* Non-blocking payment APIs
* Distributed background workers
* Reliable webhook delivery
* Secure merchant integrations
* Fault-tolerant async workflows

The project is designed for **production realism**, **evaluation automation**, and **system-design assessment**.


