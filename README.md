# Order Processing Workflow - Robust with Error Alerting & Retries

A production-ready n8n automation workflow for processing customer orders with payment processing, inventory management, and advanced error handling capabilities.

## 📋 Overview

This workflow handles complete order processing lifecycle including:
- Order receipt and data extraction
- Idempotency key generation (SHA-256) to prevent duplicate processing
- Payment gateway integration with automatic retry logic
- Inventory management updates
- Comprehensive error alerting and fallback responses
- Webhook-based communication

## ✨ Key Features

### 1. **Idempotency Protection**
- Generates SHA-256 based idempotency keys from order data
- Prevents duplicate order processing
- Returns cached response for duplicate requests

### 2. **Retry Mechanism**
- Automatic retry for failed payment requests (up to 3 attempts)
- Configurable delay between retries
- Smart fallback on max retry exhaustion

### 3. **Error Handling**
- Comprehensive error catching and logging
- Structured error responses (HTTP 500)
- User-friendly error messages

### 4. **Data Processing**
- Extracts order details: orderId, amount, items, customerId
- Timestamps all operations for audit trail
- Maintains order status throughout processing

## 🔄 Workflow Flow

```
Webhook Input (Order Received)
    ↓
Extract Order Data (Set node)
    ↓
Generate Idempotency Key (Code node)
    ↓
Check for Duplicates (IF condition)
    ├→ [Duplicate Found] → Return 200 (cached response)
    └→ [New Order] → Continue
         ↓
    Process Payment (HTTP Request)
         ↓
    Update Inventory (HTTP Request)
         ↓
    Build Success Response
         ↓
    Return 200 (Success)

[On Payment Failure]
    ↓
Wait (Retry Delay)
    ↓
Check Retry Count < 3
    ├→ [Yes] → Retry Payment
    └→ [No] → Build Error Response → Return 500
```

## 📥 Input Format

The webhook expects a JSON payload with the following structure:

```json
{
  "orderId": "ORD-12345",
  "amount": 99.99,
  "customerId": "CUST-456",
  "items": [
    {
      "productId": "PROD-789",
      "quantity": 2,
      "price": 49.99
    }
  ]
}
```

## 📤 Output Response

### Success (200)
```json
{
  "status": "success",
  "message": "Order processed successfully",
  "orderId": "ORD-12345",
  "idempotencyKey": "sha256hash...",
  "paymentStatus": "completed",
  "inventoryStatus": "updated"
}
```

### Duplicate (200)
```json
{
  "status": "success",
  "message": "Order already processed",
  "orderId": "ORD-12345"
}
```

### Error (500)
```json
{
  "status": "error",
  "message": "Order could not be processed at this time. Please try later.",
  "orderId": "ORD-12345",
  "error": "error details",
  "statusCode": 500,
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

## 🔧 Configuration

### Nodes Configuration

| Node | Type | Purpose |
|------|------|---------|
| Webhook: Order Received | Webhook | Receives incoming orders via HTTP POST |
| Set: Extract Order Data | Set | Extracts and validates order information |
| Code: Generate Idempotency Key | Code | Creates SHA-256 hash for duplicate detection |
| IF: Check Duplicate | IF | Checks if order was already processed |
| HTTP Request: Payment Gateway | HTTP | Sends payment to payment processor |
| HTTP Request: Inventory Management | HTTP | Updates inventory after payment success |
| Wait: Retry Delay | Wait | Adds delay before retry attempts |
| IF: Retry Count < 3 | IF | Checks remaining retry attempts |
| Set: Build Success Response | Set | Prepares success response |
| Set: Build Fallback Response | Set | Prepares error response |
| Set: Duplicate Response | Set | Prepares duplicate response |
| Webhook Response nodes | Webhook Response | Returns responses to client |

### External APIs Required

1. **Payment Gateway API**
   - Endpoint: `https://https-payment-random-beeceptor-com.free.beeceptor.com`
   - Method: POST
   - Parameters: amount, orderId, idempotencyKey

2. **Inventory Management API**
   - Endpoint: `https://https-payment-random-beeceptor-com.free.beeceptor.com`
   - Method: POST
   - Parameters: items, orderId, idempotencyKey

## 🚀 Usage

1. **Deploy the workflow** in your n8n instance
2. **Activate the workflow** (currently set to `active: false`)
3. **Get the webhook URL** from the "Webhook: Order Received" node
4. **Send POST requests** to the webhook with order data
5. **Handle responses** according to status codes (200 for success, 500 for errors)

## 📊 Data Flow

```
Input Validation → Idempotency Check → Payment Processing → 
Inventory Update → Response Building → Webhook Response
```

## ⚠️ Error Scenarios

| Scenario | Behavior |
|----------|----------|
| Duplicate Order | Returns 200 with "Order already processed" |
| Payment Fails (Retry < 3) | Waits then retries |
| Payment Fails (Retry = 3) | Returns 500 with error message |
| Invalid Input | Returns 500 with error details |
| Inventory Update Fails | Returns 500 |

## 🔐 Security Features

- **Idempotency Keys**: Prevents duplicate processing using SHA-256 hashing
- **Error Messages**: Sanitized to avoid exposing internal details
- **Status Codes**: Appropriate HTTP status codes for different scenarios
- **Timestamp Logging**: All operations timestamped for audit trail

## 🛠️ Customization Options

- Adjust retry count in "IF: Retry Count < 3" node
- Modify wait duration in "Wait: Retry Delay" node
- Update API endpoints for your payment/inventory services
- Customize response messages in Set nodes
- Add additional validation in Code nodes

## 📝 Workflow Metadata

- **Workflow ID**: SrCSIKyGNRBcFe0v
- **Instance ID**: a8385e563e5c693c65213c0e9ef5f736ebb5ea263f9cb095dbc5a19e2c828398
- **Execution Order**: v1
- **Binary Mode**: Separate
- **Status**: Inactive (Requires activation before use)

## 👥 Author & Contributors

- Workflow Designer: [Your Name/Team]
- Last Updated: 2024
- Version: 1.0

## 📞 Support

For issues or questions about this workflow:
- Check the error logs in n8n
- Verify API endpoint connectivity
- Ensure idempotency keys are being generated correctly
- Review webhook payload format

---

**Note**: This workflow is designed for production use with proper error handling and retry mechanisms. Ensure all external APIs are properly configured before activation.
