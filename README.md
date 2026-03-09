# othoba-connect
A wholesale partner integration layer that synchronizes product information and inventory levels with the **NOP Commerce** platform via a secure REST API.

---

## Overview

**othoba-connect** exposes the Partners API (`/api/partners`) so wholesale suppliers can:

- **Sync product information** – names, MRP, and wholesale pricing in real-time
- **Update inventory levels** – stock quantities and availability status automatically
- **Track changes** – every operation produces an audit trail visible in the admin panel within seconds

**Base URL:** `https://connect.othoba.com/api/partners`  
**Authentication:** API Key (`Authorization: API-Key <your-key>`)  
**Content-Type:** `application/json`

---

## Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| `POST` | `/api/partners/update-product` | Sync product name and pricing |
| `POST` | `/api/partners/update-stock` | Sync inventory quantity and availability |

### Update Product – minimal example

```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "12345",
    "name": "Premium Product Name",
    "sku": "PROD-001",
    "mrp_price": 299.99,
    "wholesale_price": 199.99
  }'
```

### Update Stock – minimal example

```bash
curl -X POST https://connect.othoba.com/api/partners/update-stock \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "12345",
    "sku": "PROD-001",
    "stock": 500
  }'
```

---

## Authentication

All requests require the `Authorization` header:

```
Authorization: API-Key YOUR_API_KEY_HERE
```

Contact your account administrator to obtain an API key. Store it securely (environment variable or secrets manager).

---

## Status Codes

| Code | Meaning |
|------|---------|
| `200 OK` | Request processed successfully |
| `400 Bad Request` | Validation or operation error |
| `401 Unauthorized` | Missing or invalid API key |
| `500 Internal Server Error` | Unexpected server error |

---

## Documentation

| File | Description |
|------|-------------|
| [PARTNERS_API_QUICKSTART.md](PARTNERS_API_QUICKSTART.md) | Get up and running in 5 minutes |
| [PARTNERS_API_DOCUMENTATION.md](PARTNERS_API_DOCUMENTATION.md) | Full endpoint reference, request/response models, error handling, and business logic |
| [PARTNERS_API_INTEGRATION_GUIDE.md](PARTNERS_API_INTEGRATION_GUIDE.md) | Step-by-step integration patterns in C#, Python, and Node.js |
| [PARTNERS_API_OPENAPI.yaml](PARTNERS_API_OPENAPI.yaml) | OpenAPI 3.x specification for code generation and tooling |

---

## Quick Reference

### Required fields – Update Product

| Field | Type | Description |
|-------|------|-------------|
| `product_id` | `string` | Unique product identifier |
| `name` | `string` | Product name/title |
| `mrp_price` | `decimal` | Maximum Retail Price (> 0) |
| `wholesale_price` | `decimal` | Cost price (≥ 0) |

### Required fields – Update Stock

| Field | Type | Description |
|-------|------|-------------|
| `product_id` | `string` | Unique product identifier |
| `stock` | `integer` | Quantity in stock (≥ 0). Setting `0` marks the product **Out of Stock** |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **401 Unauthorized** | Verify the API key is correct and present in the `Authorization` header |
| **400 Bad Request** | Ensure all required fields are included and correctly formatted |
| **Product Not Found** | Confirm `product_id` exists in the platform |
| **Invalid Price** | `mrp_price` must be > 0; `wholesale_price` must be ≥ 0 |

---

## License

See repository root for license information.