# Partners API - Quick Start Guide

Get up and running with the Partners API in 5 minutes!

## Prerequisites

- Valid API Key from your account administrator
- Basic understanding of HTTP/REST APIs
- Tools: cURL, Postman, or any HTTP client

---

## Step 1: Get Your API Key

Contact your administrator to obtain your API Key. You'll need this for all requests.

---

## Step 2: Test Your Connection

Send a test request to verify your API key works:

```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -d '{
    "product_id": "TEST-001",
    "name": "Test Product",
    "sku": "TEST-SKU",
    "mrp_price": 100,
    "wholesale_price": 50
  }'
```

**Expected Response (Success):**
```json
{
  "success": true,
  "message": "Product updated successfully"
}
```

---

## Step 3: Update Product Information

Use the **Update Product** endpoint to sync product data:

```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -d '{
    "product_id": "12345",
    "supplier": "Your Supplier Name",
    "name": "Product Name",
    "sku": "PROD-SKU-001",
    "mrp_price": 999.99,
    "wholesale_price": 599.99
  }'
```

**Required Fields:**
- `product_id` - Product identifier
- `name` - Product name
- `mrp_price` - Retail price (> 0)
- `wholesale_price` - Cost price (≥ 0)

---

## Step 4: Update Stock Quantities

Use the **Update Stock** endpoint to sync inventory:

```bash
curl -X POST https://connect.othoba.com/api/partners/update-stock \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -d '{
    "product_id": "12345",
    "sku": "PROD-SKU-001",
    "supplier": "Your Supplier Name",
    "stock": 500
  }'
```

**Required Fields:**
- `product_id` - Product identifier
- `stock` - Quantity in stock (≥ 0)

---

## Common Responses

### ✅ Success (200 OK)
```json
{
  "success": true,
  "message": "Product updated successfully"
}
```

### ❌ Validation Error (400 Bad Request)
```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "Name": ["'Name' must not be empty."]
    }
  }
}
```

### 🔐 Unauthorized (401)
```json
{
  "success": false,
  "message": "Invalid or missing API key"
}
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **401 Unauthorized** | Check your API key is correct and included in header |
| **400 Bad Request** | Verify all required fields are present and properly formatted |
| **Product Not Found** | Confirm product_id exists in the system |
| **Invalid Price** | Ensure mrp_price > 0 and wholesale_price ≥ 0 |

---

## What Happens Next?

**After Product Update:**
- Product name, MRP, and wholesale price are updated
- Audit trail created automatically
- Changes visible in admin panel within seconds

**After Stock Update:**
- Inventory quantity updated immediately
- If stock = 0: Product marked as "Out of Stock"
- If stock > 0: Product marked as "In Stock"
- Availability automatically updated in catalog

---

## Need More Help?

📖 **Full Documentation:** See `PARTNERS_API_DOCUMENTATION.md`

🔗 **Integration Guide:** See `PARTNERS_API_INTEGRATION_GUIDE.md`

📋 **API Specification:** See `PARTNERS_API_OPENAPI.yaml`

---

## Tips for Success

✅ **Batch Operations:** You can make multiple requests in sequence  
✅ **Error Handling:** Always check the `success` field in responses  
✅ **Logging:** All operations are logged for audit trails  
✅ **Testing:** Use the sandbox mode for testing without affecting production  

---

**Ready to integrate?** Check out the Integration Guide for detailed implementation steps!
