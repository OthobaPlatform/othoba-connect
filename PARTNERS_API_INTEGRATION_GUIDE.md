# Partners API - Integration Guide

Complete step-by-step guide for integrating the Partners API into your wholesale partner system.

## Table of Contents

1. [Overview](#overview)
2. [Authentication Setup](#authentication-setup)
3. [Implementation Approaches](#implementation-approaches)
4. [Integration Patterns](#integration-patterns)
5. [Error Handling](#error-handling)
6. [Testing Strategy](#testing-strategy)
7. [Production Deployment](#production-deployment)
8. [Monitoring & Maintenance](#monitoring--maintenance)

---

## Overview

The Partners API allows you to:
- **Sync product information** (names, pricing) in real-time
- **Update inventory levels** automatically
- **Track changes** via audit trails
- **Test safely** using sandbox mode

---

## Authentication Setup

### Step 1: Obtain API Key

1. Contact your account administrator
2. Request API key for partner integration
3. Store securely (environment variable, secrets manager, etc.)

### Step 2: Include in Requests

All requests require the `Authorization` header:

```
Authorization: API-Key YOUR_API_KEY_HERE
```

### Example with cURL
```bash
curl -X POST https://connect.othoba.com/api/partners/update-product \
  -H "Authorization: API-Key YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### Example with C#
```csharp
using System.Net.Http;

var client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", "API-Key YOUR_API_KEY_HERE");

var response = await client.PostAsJsonAsync(
    "https://connect.othoba.com/api/partners/update-product",
    request
);
```

### Example with Python
```python
import requests

headers = {
    "Authorization": "API-Key YOUR_API_KEY_HERE",
    "Content-Type": "application/json"
}

response = requests.post(
    "https://connect.othoba.com/api/partners/update-product",
    json=payload,
    headers=headers
)
```

### Example with Node.js
```javascript
const fetch = require('node-fetch');

const response = await fetch('https://connect.othoba.com/api/partners/update-product', {
  method: 'POST',
  headers: {
    'Authorization': 'API-Key YOUR_API_KEY_HERE',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(payload)
});
```

---

## Implementation Approaches

### Approach 1: Direct HTTP Calls (Lightweight)

**Best for:** Simple scripts, webhooks, small-scale integrations

```csharp
public class PartnerApiClient
{
    private readonly string _apiKey;
    private readonly HttpClient _httpClient;
    private readonly string _baseUrl = "https://connect.othoba.com";

    public PartnerApiClient(string apiKey)
    {
        _apiKey = apiKey;
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("Authorization", $"API-Key {apiKey}");
    }

    public async Task<bool> UpdateProductAsync(ProductUpdateRequest request)
    {
        var response = await _httpClient.PostAsJsonAsync(
            $"{_baseUrl}/api/partners/update-product",
            request
        );

        return response.IsSuccessStatusCode;
    }

    public async Task<bool> UpdateStockAsync(StockUpdateRequest request)
    {
        var response = await _httpClient.PostAsJsonAsync(
            $"{_baseUrl}/api/partners/update-stock",
            request
        );

        return response.IsSuccessStatusCode;
    }
}
```

### Approach 2: SDK/Wrapper (Recommended)

**Best for:** Production systems, complex integrations, error handling

```csharp
public class PartnersApiSdkClient
{
    private readonly HttpClient _httpClient;
    private readonly string _baseUrl;
    private readonly ILogger _logger;

    public PartnersApiSdkClient(string apiKey, string baseUrl, ILogger logger)
    {
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("Authorization", $"API-Key {apiKey}");
        _httpClient.DefaultRequestHeaders.Add("User-Agent", "PartnerIntegration/1.0");
        _baseUrl = baseUrl;
        _logger = logger;
    }

    public async Task UpdateProductAsync(ProductUpdateRequest request)
    {
        try
        {
            _logger.LogInformation("Updating product: {ProductId}", request.ProductId);

            var response = await _httpClient.PostAsJsonAsync(
                $"{_baseUrl}/api/partners/update-product",
                request
            );

            if (!response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                _logger.LogError("Product update failed: {StatusCode} - {Content}", 
                    response.StatusCode, content);
                throw new ApiException(response.StatusCode, content);
            }

            _logger.LogInformation("Product updated successfully: {ProductId}", request.ProductId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating product: {ProductId}", request.ProductId);
            throw;
        }
    }

    public async Task UpdateStockAsync(StockUpdateRequest request)
    {
        try
        {
            _logger.LogInformation("Updating stock: {ProductId} - Qty: {Quantity}", 
                request.ProductId, request.Quantity);

            var response = await _httpClient.PostAsJsonAsync(
                $"{_baseUrl}/api/partners/update-stock",
                request
            );

            if (!response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                _logger.LogError("Stock update failed: {StatusCode} - {Content}", 
                    response.StatusCode, content);
                throw new ApiException(response.StatusCode, content);
            }

            _logger.LogInformation("Stock updated successfully: {ProductId} - Qty: {Quantity}", 
                request.ProductId, request.Quantity);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating stock: {ProductId}", request.ProductId);
            throw;
        }
    }
}

public class ApiException : Exception
{
    public int StatusCode { get; }

    public ApiException(int statusCode, string message) : base(message)
    {
        StatusCode = statusCode;
    }
}
```

### Approach 3: Scheduled Background Job

**Best for:** Daily/hourly syncs from inventory management system

```csharp
public class PartnerInventorySyncJob
{
    private readonly PartnersApiSdkClient _apiClient;
    private readonly IInventoryService _inventoryService;
    private readonly ILogger _logger;

    public PartnerInventorySyncJob(
        PartnersApiSdkClient apiClient,
        IInventoryService inventoryService,
        ILogger logger)
    {
        _apiClient = apiClient;
        _inventoryService = inventoryService;
        _logger = logger;
    }

    public async Task ExecuteAsync()
    {
        _logger.LogInformation("Starting inventory sync job");

        try
        {
            // Get products that need sync
            var productsToSync = await _inventoryService.GetProductsToSyncAsync();

            int successCount = 0;
            int failureCount = 0;

            foreach (var product in productsToSync)
            {
                try
                {
                    // Update product info
                    var productRequest = new ProductUpdateRequest
                    {
                        ProductId = product.Id.ToString(),
                        Name = product.Name,
                        Sku = product.Sku,
                        MrpPrice = product.MrpPrice,
                        WholesalePrice = product.WholesalePrice,
                        Supplier = "YourSupplierName"
                    };

                    await _apiClient.UpdateProductAsync(productRequest);

                    // Update stock
                    var stockRequest = new StockUpdateRequest
                    {
                        ProductId = product.Id.ToString(),
                        Sku = product.Sku,
                        Quantity = product.CurrentStock,
                        Supplier = "YourSupplierName"
                    };

                    await _apiClient.UpdateStockAsync(stockRequest);

                    successCount++;

                    // Mark as synced
                    await _inventoryService.MarkAsSyncedAsync(product.Id);
                }
                catch (Exception ex)
                {
                    failureCount++;
                    _logger.LogError(ex, "Failed to sync product: {ProductId}", product.Id);
                    // Continue with next product
                }
            }

            _logger.LogInformation(
                "Inventory sync completed. Success: {SuccessCount}, Failures: {FailureCount}",
                successCount, failureCount);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Inventory sync job failed");
            throw;
        }
    }
}
```

### Approach 4: Event-Driven Updates

**Best for:** Real-time syncs when inventory changes

```csharp
public class InventoryChangeEventHandler
{
    private readonly PartnersApiSdkClient _apiClient;
    private readonly ILogger _logger;

    public InventoryChangeEventHandler(PartnersApiSdkClient apiClient, ILogger logger)
    {
        _apiClient = apiClient;
        _logger = logger;
    }

    public async Task OnProductPriceChanged(ProductPriceChangedEvent @event)
    {
        _logger.LogInformation("Product price changed: {ProductId}", @event.ProductId);

        var request = new ProductUpdateRequest
        {
            ProductId = @event.ProductId,
            Name = @event.ProductName,
            Sku = @event.Sku,
            MrpPrice = @event.NewMrpPrice,
            WholesalePrice = @event.NewWholesalePrice,
            Supplier = @event.Supplier
        };

        await _apiClient.UpdateProductAsync(request);
    }

    public async Task OnStockLevelChanged(StockLevelChangedEvent @event)
    {
        _logger.LogInformation("Stock level changed: {ProductId}", @event.ProductId);

        var request = new StockUpdateRequest
        {
            ProductId = @event.ProductId,
            Sku = @event.Sku,
            Quantity = @event.NewQuantity,
            Supplier = @event.Supplier
        };

        await _apiClient.UpdateStockAsync(request);
    }
}
```

---

## Integration Patterns

### Pattern 1: Request-Response Synchronous

Use when you need immediate confirmation of updates:

```csharp
public async Task SyncProductImmediately(int productId)
{
    var product = await _productService.GetProductAsync(productId);
    
    var request = new ProductUpdateRequest
    {
        ProductId = product.Id.ToString(),
        Name = product.Name,
        Sku = product.Sku,
        MrpPrice = product.Price,
        WholesalePrice = product.Cost
    };

    try
    {
        await _apiClient.UpdateProductAsync(request);
        // Product successfully synced
    }
    catch (ApiException ex)
    {
        // Handle error immediately
        _logger.LogError(ex, "Failed to sync product");
        // Retry or notify user
    }
}
```

### Pattern 2: Batch Processing

Use for high-volume updates:

```csharp
public async Task SyncProductBatchAsync(List<int> productIds)
{
    var batchSize = 100;
    var batches = productIds
        .Chunk(batchSize)
        .ToList();

    foreach (var batch in batches)
    {
        var tasks = batch.Select(productId =>
        {
            var product = _productService.GetProductAsync(productId);
            var request = new ProductUpdateRequest
            {
                ProductId = product.Id.ToString(),
                Name = product.Name,
                Sku = product.Sku,
                MrpPrice = product.Price,
                WholesalePrice = product.Cost
            };

            return _apiClient.UpdateProductAsync(request);
        });

        await Task.WhenAll(tasks);
        _logger.LogInformation("Batch processed: {Count} products", batch.Length);

        // Add delay between batches to avoid throttling
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
}
```

### Pattern 3: Queue-Based Processing

Use for reliable, durable updates:

```csharp
public class ProductUpdateQueue
{
    private readonly IQueue<ProductUpdateRequest> _queue;
    private readonly PartnersApiSdkClient _apiClient;
    private readonly ILogger _logger;

    public async Task QueueProductUpdateAsync(ProductUpdateRequest request)
    {
        await _queue.EnqueueAsync(request);
        _logger.LogInformation("Product update queued: {ProductId}", request.ProductId);
    }

    public async Task ProcessQueueAsync()
    {
        while (!_queue.IsEmpty)
        {
            try
            {
                var request = await _queue.DequeueAsync(TimeSpan.FromSeconds(30));
                
                if (request != null)
                {
                    await _apiClient.UpdateProductAsync(request);
                    _logger.LogInformation("Queued product update processed: {ProductId}", request.ProductId);
                }
            }
            catch (ApiException ex)
            {
                _logger.LogError(ex, "Failed to process queued update, requeuing");
                // Re-queue failed item for retry
                await _queue.EnqueueAsync(request);
            }
        }
    }
}
```

---

## Error Handling

### Graceful Degradation

```csharp
public async Task UpdateProductWithFallback(ProductUpdateRequest request)
{
    try
    {
        await _apiClient.UpdateProductAsync(request);
    }
    catch (ApiException ex) when (ex.StatusCode == 400)
    {
        // Validation error - log and alert
        _logger.LogWarning(ex, "Validation error for product: {ProductId}", request.ProductId);
        await _notificationService.AlertValidationErrorAsync(request.ProductId);
    }
    catch (ApiException ex) when (ex.StatusCode == 401)
    {
        // Authentication error - stop processing
        _logger.LogError(ex, "API key is invalid");
        throw new InvalidOperationException("API authentication failed");
    }
    catch (ApiException ex) when (ex.StatusCode >= 500)
    {
        // Server error - retry with exponential backoff
        _logger.LogWarning(ex, "Server error, retrying");
        await RetryWithBackoffAsync(() => _apiClient.UpdateProductAsync(request));
    }
    catch (HttpRequestException ex)
    {
        // Network error - queue for later retry
        _logger.LogWarning(ex, "Network error, queuing for retry");
        await _queue.EnqueueAsync(request);
    }
}

private async Task RetryWithBackoffAsync(Func<Task> operation, int maxRetries = 3)
{
    for (int attempt = 0; attempt < maxRetries; attempt++)
    {
        try
        {
            await operation();
            return;
        }
        catch (Exception ex) when (attempt < maxRetries - 1)
        {
            var delay = TimeSpan.FromSeconds(Math.Pow(2, attempt));
            _logger.LogInformation("Retry {Attempt} after {Delay}ms", attempt + 1, delay.TotalMilliseconds);
            await Task.Delay(delay);
        }
    }
}
```

---

## Testing Strategy

### Unit Testing

```csharp
[TestClass]
public class PartnersApiClientTests
{
    private PartnersApiSdkClient _client;
    private Mock<HttpMessageHandler> _httpHandlerMock;

    [TestInitialize]
    public void Setup()
    {
        _httpHandlerMock = new Mock<HttpMessageHandler>();
        var httpClient = new HttpClient(_httpHandlerMock.Object);
        _client = new PartnersApiSdkClient("test-key", "https://connect.othoba.com", httpClient);
    }

    [TestMethod]
    public async Task UpdateProduct_WithValidRequest_ReturnsSuccess()
    {
        // Arrange
        var request = new ProductUpdateRequest
        {
            ProductId = "123",
            Name = "Test Product",
            Sku = "TEST-001",
            MrpPrice = 100,
            WholesalePrice = 50
        };

        _httpHandlerMock
            .Setup(h => h.SendAsync(It.IsAny<HttpRequestMessage>(), It.IsAny<CancellationToken>()))
            .ReturnsAsync(new HttpResponseMessage { StatusCode = HttpStatusCode.OK });

        // Act
        await _client.UpdateProductAsync(request);

        // Assert
        _httpHandlerMock.Verify(h => h.SendAsync(It.IsAny<HttpRequestMessage>(), It.IsAny<CancellationToken>()), Times.Once);
    }

    [TestMethod]
    [ExpectedException(typeof(ApiException))]
    public async Task UpdateProduct_WithInvalidRequest_ThrowsException()
    {
        // Arrange
        var request = new ProductUpdateRequest { ProductId = "123" }; // Missing required fields

        _httpHandlerMock
            .Setup(h => h.SendAsync(It.IsAny<HttpRequestMessage>(), It.IsAny<CancellationToken>()))
            .ReturnsAsync(new HttpResponseMessage { StatusCode = HttpStatusCode.BadRequest });

        // Act
        await _client.UpdateProductAsync(request);
    }
}
```

### Integration Testing

```csharp
[TestClass]
public class PartnersApiIntegrationTests
{
    private readonly string _apiKey = Environment.GetEnvironmentVariable("PARTNERS_API_KEY_TEST");
    private readonly string _apiBaseUrl = "https://api-test.example.com";
    private PartnersApiSdkClient _client;

    [TestInitialize]
    public void Setup()
    {
        _client = new PartnersApiSdkClient(_apiKey, _apiBaseUrl);
    }

    [TestMethod]
    public async Task UpdateProduct_EndToEnd_Success()
    {
        // Arrange
        var request = new ProductUpdateRequest
        {
            ProductId = "TEST-" + DateTime.Now.Ticks,
            Name = "Integration Test Product",
            Sku = "TEST-SKU-" + DateTime.Now.Ticks,
            MrpPrice = 999.99m,
            WholesalePrice = 499.99m
        };

        // Act
        await _client.UpdateProductAsync(request);

        // Assert - Product should exist in system
        var product = await _apiClient.GetProductAsync(request.ProductId);
        Assert.IsNotNull(product);
        Assert.AreEqual("Integration Test Product", product.Name);
    }
}
```

### Sandbox Testing

Use sandbox mode before production:

```csharp
public async Task TestInSandbox()
{
    var request = new StockUpdateRequest
    {
        ProductId = "123",
        Sku = "TEST-SKU",
        Quantity = 0  // Will set product to out-of-stock
    };

    // This will validate but NOT write to database in sandbox mode
    await _apiClient.UpdateStockAsync(request);

    // Verify the request was valid
    // No actual database changes were made
}
```

---

## Production Deployment

### Pre-Deployment Checklist

- [ ] API key stored in secure configuration (not in code)
- [ ] Error handling implemented for all scenarios
- [ ] Logging configured and tested
- [ ] Rate limiting strategy defined
- [ ] Retry logic with exponential backoff implemented
- [ ] Database connection pooling optimized
- [ ] SSL/TLS certificate validation enabled
- [ ] Integration tests passing
- [ ] Load testing completed
- [ ] Documentation updated

### Configuration Management

```csharp
// appsettings.json
{
  "PartnersApi": {
    "BaseUrl": "https://connect.othoba.com",
    "ApiKey": "${PARTNERS_API_KEY}",  // Use environment variable
    "Timeout": "30",
    "MaxRetries": 3,
    "BatchSize": 100
  }
}

// Startup
services.Configure<PartnersApiSettings>(configuration.GetSection("PartnersApi"));
services.AddScoped(sp =>
{
    var settings = sp.GetRequiredService<IOptions<PartnersApiSettings>>();
    return new PartnersApiSdkClient(
        settings.Value.ApiKey,
        settings.Value.BaseUrl,
        sp.GetRequiredService<ILogger<PartnersApiSdkClient>>()
    );
});
```

### Deployment Steps

1. **Prepare Environment**
   ```bash
   # Set environment variables
   export PARTNERS_API_KEY=your_production_key
   export PARTNERS_API_URL=https://connect.othoba.com
   ```

2. **Deploy Application**
   ```bash
   dotnet publish -c Release
   # Deploy to production
   ```

3. **Verify Integration**
   ```bash
   # Run health check
   curl -H "Authorization: API-Key $PARTNERS_API_KEY" \
     https://connect.othoba.com/api/partners/update-product -X POST
   ```

4. **Monitor**
   - Watch logs for errors
   - Monitor API response times
   - Track success/failure rates

---

## Monitoring & Maintenance

### Key Metrics to Monitor

```csharp
public class PartnerApiMetrics
{
    // Request metrics
    public long TotalRequests { get; set; }
    public long SuccessfulRequests { get; set; }
    public long FailedRequests { get; set; }

    // Performance metrics
    public double AverageResponseTime { get; set; }
    public double P95ResponseTime { get; set; }
    public double P99ResponseTime { get; set; }

    // Error metrics
    public Dictionary<int, long> ErrorsByStatusCode { get; set; }
    public long ValidationErrors { get; set; }
    public long AuthenticationErrors { get; set; }

    public double SuccessRate => (double)SuccessfulRequests / TotalRequests;
}
```

### Health Checks

```csharp
public class PartnersApiHealthCheck : IHealthCheck
{
    private readonly PartnersApiSdkClient _apiClient;

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Make a test request
            var testRequest = new ProductUpdateRequest
            {
                ProductId = "health-check-" + DateTime.UtcNow.Ticks,
                Name = "Health Check",
                Sku = "HEALTH-CHECK",
                MrpPrice = 1,
                WholesalePrice = 0.5m
            };

            await _apiClient.UpdateProductAsync(testRequest);

            return HealthCheckResult.Healthy("Partners API is accessible");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Partners API health check failed", ex);
        }
    }
}

// In Startup
services.AddHealthChecks()
    .AddCheck<PartnersApiHealthCheck>("partners-api", tags: new[] { "api" });
```

### Logging Best Practices

```csharp
// Log important events
_logger.LogInformation("Product sync started for {Count} products", productCount);
_logger.LogInformation("Product {ProductId} synced successfully", productId);
_logger.LogWarning("Product {ProductId} sync failed, will retry", productId);
_logger.LogError(ex, "Critical error syncing product {ProductId}", productId);

// Structured logging for analysis
using (_logger.BeginScope(new Dictionary<string, object>
{
    ["ProductId"] = productId,
    ["Operation"] = "Sync",
    ["Timestamp"] = DateTime.UtcNow
}))
{
    await _apiClient.UpdateProductAsync(request);
}
```

---

## Support Resources

📖 **Full API Documentation:** `PARTNERS_API_DOCUMENTATION.md`

⚡ **Quick Start:** `PARTNERS_API_QUICKSTART.md`

📋 **OpenAPI Specification:** `PARTNERS_API_OPENAPI.yaml`

---

**Questions?** Contact your API administrator or refer to the troubleshooting section in the main documentation.
