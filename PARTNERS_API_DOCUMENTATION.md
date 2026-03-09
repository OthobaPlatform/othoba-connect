# Partners API Documentation

## Overview

The Partners API provides endpoints for wholesale partners to synchronize product information and inventory levels with the NOP commerce platform. This integration enables real-time product data updates and stock quantity management from external wholesale suppliers.

**Base URL:** `/api/partners`

**Authentication:** API Key (via `[ApiKey]` attribute)

**Content Type:** `application/json`

---

## Table of Contents

1. [Authentication](#authentication)
2. [Endpoints](#endpoints)
   - [Update Product](#update-product)
   - [Update Stock](#update-stock)
3. [Request/Response Models](#requestresponse-models)
4. [Error Handling](#error-handling)
5. [Examples](#examples)
6. [Business Logic](#business-logic)
7. [Configuration](#configuration)

---

## Authentication

All endpoints require an API Key for authorization. The API Key should be included in the request headers using the `[ApiKey]` authentication scheme.

**Authentication Scheme:** API Key Validation

**Status Codes:**
- `200 OK` - Request processed successfully
- `400 Bad Request` - Invalid request data or validation error
- `401 Unauthorized` - Missing or invalid API key
- `500 Internal Server Error` - Unexpected server error

---

## Endpoints

### Update Product

Synchronizes product information (name, pricing) with the catalog database.

**Request:**
```http
POST /api/partners/update-product
Content-Type: application/json
Authorization: API-Key [your-api-key]
```

**Method:** `POST`  
**Route:** `update-product`  
**Content-Type:** `application/json`

**Request Body:**
```json
{
  "product_id": "12345",
  "supplier": "EuroAsia Wholesale",
  "name": "Premium Product Name",
  "sku": "PROD-001",
  "mrp_price": 299.99,
  "wholesale_price": 199.99
}
```

**Response (Success - 200 OK):**
```json
{
  "success": true,
  "message": "Product updated successfully"
}
```

**Response (Validation Error - 400 Bad Request):**
```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/partners/update-product",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "Name": ["'Name' must not be empty."],
      "MrpPrice": ["'Mrp Price' must be greater than '0'."]
    }
  }
}
```

**Response (Operation Error - 400 Bad Request):**
```json
{
  "type": "https://connect.othoba.com/api/operation-error",
  "title": "Operation Error",
  "status": 400,
  "detail": "Product Not found",
  "instance": "/api/partners/update-product",
  "response": {
    "success": false,
    "message": "Product Not found"
  }
}
```

**Response (Server Error - 500 Internal Server Error):**
```json
{
  "type": "https://connect.othoba.com/api/server-error",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "An unexpected error occurred while processing the request",
  "instance": "/api/partners/update-product",
  "response": {
    "success": false,
    "message": "An unexpected error occurred while processing the request"
  }
}
```

---

### Update Stock

Synchronizes inventory levels and product availability status with the catalog database.

**Request:**
```http
POST /api/partners/update-stock
Content-Type: application/json
Authorization: API-Key [your-api-key]
```

**Method:** `POST`  
**Route:** `update-stock`  
**Content-Type:** `application/json`

**Request Body:**
```json
{
  "product_id": "12345",
  "sku": "PROD-001",
  "supplier": "EuroAsia Wholesale",
  "stock": 500
}
```

**Response (Success - 200 OK):**
```json
{
  "success": true,
  "message": "Stock updated successfully"
}
```

**Response (Validation Error - 400 Bad Request):**
```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/partners/update-stock",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "Quantity": ["'Quantity' must be greater than or equal to '0'."]
    }
  }
}
```

---

## Request/Response Models

### ProductUpdateRequest

Updates product metadata including name and pricing information.

| Field | Type | Required | Description | JSON Property |
|-------|------|----------|-------------|---------------|
| `ProductId` | `string` | Yes | Unique identifier of the product | `product_id` |
| `Supplier` | `string` | No | Supplier/partner name | `supplier` |
| `Name` | `string` | Yes | Product name/title | `name` |
| `Sku` | `string` | No | Stock Keeping Unit code | `sku` |
| `MrpPrice` | `decimal` | Yes | Maximum Retail Price | `mrp_price` |
| `WholesalePrice` | `decimal` | Yes | Wholesale/cost price | `wholesale_price` |

**Validation Rules:**
- `ProductId`: Required, non-null/empty
- `Name`: Required, non-null/empty
- `MrpPrice`: Required, must be greater than 0
- `WholesalePrice`: Required, must be greater than or equal to 0

---

### StockUpdateRequest

Updates inventory levels and product availability status.

| Field | Type | Required | Description | JSON Property |
|-------|------|----------|-------------|---------------|
| `ProductId` | `string` | Yes | Unique identifier of the product | `product_id` |
| `Sku` | `string` | No | Stock Keeping Unit code | `sku` |
| `Supplier` | `string` | No | Supplier/partner name | `supplier` |
| `Quantity` | `int` | Yes | Available stock quantity | `stock` |

**Validation Rules:**
- `ProductId`: Required, non-null/empty
- `Quantity`: Required, must be greater than or equal to 0

**Availability Logic:**
- **Stock > 0**: 
  - `DisableBuyButton = false`
  - `SoldOut = false`
  - Product is available for purchase
  
- **Stock = 0**:
  - `DisableBuyButton = true`
  - `SoldOut = true`
  - Product is marked as out of stock

---

### ApiResponse

Standard response model for all API operations.

```csharp
public class ApiResponse
{
    public bool Success { get; set; }          // Operation result
    public string Message { get; set; }        // Response message
    public Dictionary<string, string[]> Errors { get; set; }  // Validation errors
}
```

**Factory Methods:**

```csharp
// Success response
ApiResponse.SuccessResponse("Product updated successfully")

// Error response with validation errors
ApiResponse.ErrorResponse("Validation failed", new Dictionary<string, string[]> {
    { "Name", new[] { "'Name' must not be empty." } }
})

// Error response without details
ApiResponse.ErrorResponse("An error occurred during the operation.")
```

---

## Error Handling

The API uses HTTP status codes and Problem Details format (RFC 7807) for error responses.

### Error Status Codes

| Status Code | Type | Description |
|------------|------|-------------|
| `200` | ✅ Success | Request processed successfully |
| `400` | ❌ Bad Request | Invalid input, validation error, or operation failure |
| `401` | 🔐 Unauthorized | Missing or invalid API key |
| `500` | ⚠️ Server Error | Unexpected server error |

### Problem Details Response Format

All error responses use the standard Problem Details format:

```json
{
  "type": "https://connect.othoba.com/api/[error-type]",
  "title": "[Error Title]",
  "status": 400,
  "detail": "[Detailed error message]",
  "instance": "[Request path]",
  "response": {
    "success": false,
    "message": "[API response message]",
    "errors": { /* validation errors if applicable */ }
  }
}
```

### Error Types

1. **Validation Error** (`https://connect.othoba.com/api/validation-error`)
   - HTTP 400
   - Validation rule violations
   - Includes detailed field-level errors

2. **Operation Error** (`https://connect.othoba.com/api/operation-error`)
   - HTTP 400
   - Business logic violations
   - Examples: Product not found, API disabled, invalid state

3. **Server Error** (`https://connect.othoba.com/api/server-error`)
   - HTTP 500
   - Unexpected exceptions
   - Generic error message for security

---

## Examples

### Example 1: Update Product Information

**Request:**
```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY" \
  -d '{
    "product_id": "PROD-2024-001",
    "supplier": "EuroAsia Wholesale",
    "name": "Premium Cotton T-Shirt",
    "sku": "TSHIRT-COTTON-001",
    "mrp_price": 499.99,
    "wholesale_price": 299.99
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Product updated successfully"
}
```

---

### Example 2: Update Stock Quantity

**Request:**
```bash
curl -X POST https://connect.othoba.com/api/partners/update-stock \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY" \
  -d '{
    "product_id": "PROD-2024-001",
    "sku": "TSHIRT-COTTON-001",
    "supplier": "EuroAsia Wholesale",
    "stock": 1250
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Stock updated successfully"
}
```

---

### Example 3: Validation Error Handling

**Request (Invalid - Missing Name):**
```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY" \
  -d '{
    "product_id": "PROD-2024-001",
    "supplier": "EuroAsia Wholesale",
    "sku": "TSHIRT-COTTON-001",
    "mrp_price": 499.99,
    "wholesale_price": 299.99
  }'
```

**Response:**
```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/partners/update-product",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "Name": ["'Name' must not be empty."]
    }
  }
}
```

---

### Example 4: Out of Stock State

**Request (Zero Stock):**
```bash
curl -X POST https://connect.othoba.com/api/partners/update-stock \
  -H "Content-Type: application/json" \
  -H "Authorization: API-Key YOUR_API_KEY" \
  -d '{
    "product_id": "PROD-2024-001",
    "sku": "TSHIRT-COTTON-001",
    "supplier": "EuroAsia Wholesale",
    "stock": 0
  }'
```

**Effects:**
- Product `StockQuantity` set to 0
- `DisableBuyButton` set to `true`
- `SoldOut` set to `true`
- Audit trail created in product notes

---

## Business Logic

### Product Update Workflow

```
1. ⚙️ Pre-flight Checks
   ├─ Verify API calls are enabled in StockQuantityApiSettings
   └─ Validate request is not null

2. 🔍 Validation Phase
   ├─ Execute FluentValidation on ProductUpdateRequest
   └─ Collect and return all validation errors

3. 🗃️ Database Operations
   ├─ Fetch product from catalog by item code (ProductId)
   ├─ Verify product exists
   └─ Throw InvalidOperationException if not found

4. 🧪 Sandbox Mode Check
   ├─ Check if client is sandbox/testing
   └─ Skip writes if sandbox (safe testing without production impact)

5. 📝 Apply Updates
   ├─ Update Product.Name
   ├─ Update Product.Price (MRP)
   └─ Update Product.ProductCost (Wholesale)

6. 💾 Persist Changes
   ├─ Save product updates to database
   └─ Create ProductNote audit trail

7. 📊 Logging & Response
   ├─ Log operation success
   └─ Return ApiResponse with success message
```

---

### Stock Update Workflow

```
1. ⚙️ Pre-flight Checks
   ├─ Verify API calls are enabled in StockQuantityApiSettings
   └─ Validate request is not null

2. 🔍 Validation Phase
   ├─ Execute FluentValidation on StockUpdateRequest
   └─ Collect and return all validation errors

3. 🗃️ Database Operations
   ├─ Fetch product from catalog by item code (ProductId)
   ├─ Verify product exists
   └─ Throw InvalidOperationException if not found

4. 🧪 Sandbox Mode Check
   ├─ Check if client is sandbox/testing
   └─ Skip writes if sandbox (safe testing without production impact)

5. 📈 Update Inventory
   ├─ Set Product.StockQuantity = request.Quantity
   └─ Determine availability based on quantity

6. 📈 Manage Availability Status
   ├─ If Quantity > 0:
   │  ├─ DisableBuyButton = false
   │  └─ SoldOut = false
   └─ If Quantity = 0:
      ├─ DisableBuyButton = true
      └─ SoldOut = true

7. 💾 Persist Changes
   ├─ Save inventory updates to database
   └─ Create ProductNote audit trail

8. 📊 Logging & Response
   ├─ Log operation success with quantity
   └─ Return ApiResponse with success message
```

---

### Sandbox Mode

The API supports a sandbox mode for testing purposes. When a request comes from a sandbox API client:

- **Validation:** All validation rules are executed normally
- **Database Operations:** Product fetch is performed normally
- **Writes:** Database writes are **skipped**
- **Testing:** Allows safe testing without affecting production data

**Determination:** Sandbox mode is detected via `IHttpContextAccessor.HttpContext.IsSandBoxApiClient()`

---

### Audit Trail

All product updates create a `ProductNote` record for compliance and audit purposes:

```csharp
var productNote = new ProductNote
{
    ProductId = product.Id,
    Note = $"[Operation] - Details of changes",
    CreatedOnUtc = DateTime.UtcNow
};
```

**Product Update Note Format:**
```
Product updated via Partner API - Name: {name}, MRP: {price}, Wholesale Price: {cost}
```

**Stock Update Note Format:**
```
Stock updated via Partner API - New Quantity: {quantity}
```

---

## Configuration

### StockQuantityApiSettings

The API behavior is controlled by configuration settings:

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `EnableApiCall` | `bool` | - | Global switch to enable/disable API operations |

**Behavior:**
- If `EnableApiCall = false`: All API endpoints return `InvalidOperationException` with message "API calls are currently disabled"

### Dependency Injection

Required dependencies injected into `PartnersController`:

```csharp
- IPartnerService: Business logic for product/stock updates
- ILogger<PartnersController>: Structured logging using Microsoft.Extensions.Logging
```

Required dependencies injected into `PartnerService`:

```csharp
- IValidator<ProductUpdateRequest>: FluentValidation for product requests
- IValidator<StockUpdateRequest>: FluentValidation for stock requests
- ILogger: NOP-specific async logging (Nop.Services.Logging)
- IProductService: NOP product service for catalog operations
- StockQuantityApiSettings: API configuration settings
- IHttpContextAccessor: HTTP context for sandbox detection
- IRepository<ProductNote>: EF Core repository for audit trails
```

---

## Logging

The API implements comprehensive logging at multiple levels:

### Logging Levels

| Level | When | Example |
|-------|------|---------|
| **Information** | Successful validation, operation completion | "UpdateProduct: Validated request for ProductId: 12345, SKU: PROD-001" |
| **Warning** | Validation failures, operation issues | "UpdateProduct: Validation failed for ProductId: 12345. Errors: ..." |
| **Error** | Unexpected exceptions, API disabled | "UpdateProduct: Error processing product update for ProductId: 12345" |

### Log Context

All log messages include:
- Operation name (UpdateProduct, UpdateStock)
- Product ID or relevant identifiers
- Structured context data using log scopes

---

## Performance Considerations

- **Update Product:** Typical execution 200-500ms per product
- **Update Stock:** Typical execution 150-400ms per product
- **ConfigureAwait(false):** Used for non-blocking async operations
- **No Blocking Calls:** Full async/await pattern throughout

---

## Security

1. **API Key Authentication:** All endpoints require valid API key
2. **Validation:** Comprehensive input validation via FluentValidation
3. **Sandbox Mode:** Prevents accidental production changes during testing
4. **Audit Trail:** All changes logged via ProductNote for compliance
5. **Error Responses:** Generic error messages for server errors to prevent information leakage

---

## Related Files

- **Controller:** `Plugins\Nop.Plugin.Misc.WebApi.Backend\Controllers\Partners\PartnersController.cs`
- **Service:** `Plugins\Nop.Plugin.Misc.WebApi.Backend\Services\PartnerService.cs`
- **Interface:** `Plugins\Nop.Plugin.Misc.WebApi.Backend\Services\IPartnerService.cs`
- **DTOs:** 
  - `Plugins\Nop.Plugin.Misc.WebApi.Backend\Dto\Partners\ProductUpdateRequest.cs`
  - `Plugins\Nop.Plugin.Misc.WebApi.Backend\Dto\Partners\StockUpdateRequest.cs`
  - `Plugins\Nop.Plugin.Misc.WebApi.Backend\Dto\Partners\ApiResponse.cs`
- **Validators:**
  - `Plugins\Nop.Plugin.Misc.WebApi.Backend\Validators\Partners\ProductUpdateRequestValidator.cs`
  - `Plugins\Nop.Plugin.Misc.WebApi.Backend\Validators\Partners\StockUpdateRequestValidator.cs`

---

## Support & Troubleshooting

### Common Issues

1. **401 Unauthorized**
   - Verify API key is correct and included in request headers
   - Check if API key has appropriate permissions

2. **400 Validation Error**
   - Review validation error details in response
   - Ensure all required fields are provided
   - Verify data types match expected formats

3. **404 Product Not Found**
   - Confirm ProductId exists in the system
   - Check that the product is not deleted
   - Verify ProductId format matches system requirements

4. **API calls are currently disabled**
   - Contact administrator to enable API calls
   - Check StockQuantityApiSettings configuration
   - Verify feature flags are enabled

### Debug Mode

When `EnableApiCall = false`, all endpoints will return:
```json
{
  "type": "https://connect.othoba.com/api/operation-error",
  "title": "Operation Error",
  "status": 400,
  "detail": "API calls are currently disabled",
  "instance": "/api/partners/[endpoint]"
}
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024 | Initial API implementation |
| 1.1 | 2024 | Added comprehensive documentation |

---

**Last Updated:** 2024

**Maintained By:** Development Team
