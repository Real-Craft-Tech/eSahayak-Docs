# eSahayak API Documentation v1

Welcome to the eSahayak API! This documentation will help you integrate stamp duty services into your application.

## 📚 Table of Contents

- [Getting Started](#getting-started)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [Data APIs](#data-apis)
- [Stamp Order API](#stamp-order-api)
- [Error Handling](#error-handling)
- [Rate Limits & Credits](#rate-limits--credits)
- [Examples](#examples)

## 🚀 Getting Started

The eSahayak API allows you to:
- **Get reference data** (states, districts, SROs, articles)
- **Create stamp orders** programmatically
- **Integrate stamp duty services** into your application

### Prerequisites

1. **Create an eSahayak account** at [https://esahayak.io](https://esahayak.io)
2. **Generate an API key** from your workspace settings
3. **Add credits** to your workspace for stamp orders

## 🔐 Authentication

### API Key Authentication

All API requests require authentication using your workspace API key.

**Header Required:**
```
x-api-key: YOUR_API_KEY_HERE
```

**Getting Your API Key:**
1. Log in to your eSahayak dashboard
2. Go to **Settings** → **API Access**
3. Click **Generate API Key** (or **Regenerate** if you already have one)
4. Copy the generated key

⚠️ **Important:** Keep your API key secure and never expose it in client-side code.

## 🌍 Base URL

```
https://esahayak.io/api/v1
```

All API endpoints are relative to this base URL.

## 📊 Data APIs

These APIs provide reference data needed for creating stamp orders. **No authentication required.**

### 1. States API

Get list of all available states for stamp duty services.

```http
GET /data/states
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "name": "Assam",
      "code": "AS",
      "minAmount": 100,
      "maxAmount": null,
      "cessRequired": false,
      "propertyAreaRequired": false,
      "sroRequired": true,
      "isTraditional": false
    }
  ],
  "count": 25
}
```

**Response Fields:**
- `name`: Full state name (use this for stamp orders)
- `code`: State code (use for other data APIs)
- `minAmount`: Minimum stamp amount allowed
- `maxAmount`: Maximum stamp amount allowed (null = no limit)
- `cessRequired`: Whether cess amount is mandatory
- `propertyAreaRequired`: Whether property area details are required
- `sroRequired`: Whether SRO selection is mandatory

### 2. Districts API

Get districts for a specific state.

```http
GET /data/districts?stateCode={STATE_CODE}
```

**Parameters:**
- `stateCode` (required): State code from States API (e.g., "AS", "DL")

**Example:**
```http
GET /data/districts?stateCode=AS
```

**Response:**
```json
{
  "success": true,
  "stateCode": "AS",
  "data": [
    {
      "code": "AS-BK",
      "name": "Baksa"
    },
    {
      "code": "AS-BR",
      "name": "Barpeta"
    }
  ],
  "count": 32
}
```

**Response Fields:**
- `code`: District code (use this for SROs API and stamp orders)
- `name`: District name for display

### 3. SROs API

Get Sub Registrar Offices for a specific district.

```http
GET /data/sros?stateCode={STATE_CODE}&districtCode={DISTRICT_CODE}
```

**Parameters:**
- `stateCode` (required): State code from States API
- `districtCode` (required): District code from Districts API

**Example:**
```http
GET /data/sros?stateCode=AS&districtCode=AS-MO
```

**Response:**
```json
{
  "success": true,
  "stateCode": "AS",
  "districtCode": "AS-MO",
  "data": [
    {
      "code": "AS-MRG",
      "name": "Morigaon"
    }
  ],
  "count": 1
}
```

**Response Fields:**
- `code`: SRO code (use this for stamp orders if required)
- `name`: SRO name for display

### 4. Articles API

Get article codes and descriptions for a specific state.

```http
GET /data/articles?stateCode={STATE_CODE}
```

**Parameters:**
- `stateCode` (required): State code from States API

**Example:**
```http
GET /data/articles?stateCode=AS
```

**Response:**
```json
{
  "success": true,
  "stateCode": "AS",
  "data": [
    {
      "code": "AS-RG-4",
      "description": "4 - Affidavit",
      "popular": true,
      "minimumAmount": 100,
      "allowedAmounts": null,
      "isSecondPartyNA": false
    }
  ],
  "count": 59
}
```

**Response Fields:**
- `code`: Article code (use this for stamp orders)
- `description`: Article description for display
- `popular`: Whether this is a commonly used article
- `minimumAmount`: Minimum stamp amount for this article
- `allowedAmounts`: Array of allowed amounts (null = any amount above minimum)
- `isSecondPartyNA`: Whether second party details are not applicable

## 📋 Stamp Order API

Create stamp orders programmatically with automatic cost calculation and credit deduction.

### Create Stamp Order

```http
POST /stamp-order
```

**Headers:**
```
Content-Type: application/json
x-api-key: YOUR_API_KEY_HERE
```

**Request Schema:**

```json
{
  "state": {
    "name": "string (required)",
    "district": "string (optional, required if state requires SRO)",
    "sro": "string (optional, required if state requires SRO)"
  },
  "stampAmount": "number (required, min: 1)",
  "cessAmount": "number (optional, default: 0, required if state requires cess)",
  "deliveryAddresses": [
    {
      "name": "string (required)",
      "address": "string (required)",
      "pincode": "string (required, 6 digits)",
      "phone": "string (required, 10 digits)",
      "email": "string (required, valid email)"
    }
  ],
  "firstParty": {
    "name": "string (required)",
    "phone": "string (optional, 10 digits)"
  },
  "secondParty": {
    "name": "string (optional)",
    "phone": "string (optional, 10 digits)"
  },
  "stampDutyPayer": {
    "party": "first|second (required)",
    "gender": "male|female|other (required)",
    "panCardFileUrl": "string (required, valid URL)",
    "fatherHusbandName": "string (optional, required if state requires)"
  },
  "propertyArea": {
    "unit": "SQUARE_FEET|SQUARE_METER|ACRE|HECTARE|NOT_APPLICABLE (optional)",
    "value": "number (optional, required if unit is not NOT_APPLICABLE)"
  },
  "articleCode": "string (required)",
  "purpose": "string (required, min: 10 chars, max: 100 chars)"
}
```

**Field Requirements Guide:**

| Field | How to Get | Required When |
|-------|------------|---------------|
| `state.name` | From States API `name` field | Always |
| `state.district` | From Districts API `code` field | If state has `sroRequired: true` |
| `state.sro` | From SROs API `code` field | If state has `sroRequired: true` |
| `stampAmount` | User input | Always (check min/max from States API) |
| `cessAmount` | User input | If state has `cessRequired: true` |
| `articleCode` | From Articles API `code` field | Always |
| `stampDutyPayer.panCardFileUrl` | File upload URL | Always |
| `propertyArea` | User input | If state has `propertyAreaRequired: true` |

**Example Request:**

```json
{
  "state": {
    "name": "Assam",
    "district": "AS-MO",
    "sro": "AS-MRG"
  },
  "stampAmount": 1000,
  "cessAmount": 0,
  "deliveryAddresses": [
    {
      "name": "John Doe",
      "address": "123 Main Street, Guwahati",
      "pincode": "781001",
      "phone": "9876543210",
      "email": "john@example.com"
    }
  ],
  "firstParty": {
    "name": "John Doe",
    "phone": "9876543210"
  },
  "secondParty": {
    "name": "Jane Smith",
    "phone": "9876543211"
  },
  "stampDutyPayer": {
    "party": "first",
    "gender": "male",
    "panCardFileUrl": "https://example.com/pan-card.jpg"
  },
  "propertyArea": {
    "unit": "NOT_APPLICABLE",
    "value": 0
  },
  "articleCode": "AS-RG-4",
  "purpose": "Affidavit for property verification"
}
```

**Success Response (201):**

```json
{
  "success": true,
  "orderId": "order_12345",
  "stampRequestId": "stamp_67890",
  "message": "Stamp order created successfully",
  "data": {
    "order": {
      "id": "order_12345",
      "title": "Stamp Duty (₹1000) (4 - Affidavit) Assam",
      "status": "CREATED",
      "createdAt": "2024-01-15T10:30:00Z"
    },
    "stampRequest": {
      "id": "stamp_67890",
      "state": "Assam",
      "amount": 1000,
      "status": "PENDING",
      "provider": "SHCIL"
    },
    "pricing": {
      "stampAmount": 1000,
      "cessAmount": 0,
      "discountedTotal": 1125,
      "actualTotal": 1125,
      "items": [
        {
          "type": "STAMP_AMOUNT",
          "name": "Stamp Amount",
          "price": 1000,
          "discountedPrice": 1000
        },
        {
          "type": "CONVENIENCE_FEE",
          "name": "Convenience Fee",
          "price": 100,
          "discountedPrice": 100
        },
        {
          "type": "DELIVERY_CHARGE",
          "name": "Delivery Charge",
          "price": 25,
          "discountedPrice": 25
        }
      ],
      "couponApplied": null
    },
    "credits": {
      "previousBalance": 5000,
      "amountDeducted": 1125,
      "remainingBalance": 3875
    },
    "partnerAssignment": {
      "backgroundJobResponse": {
        "ids": ["01JBEX7MH8K2QN3YN4HCB0K123"]
      }
    }
  }
}
```

## ❌ Error Handling

### Error Response Format

```json
{
  "error": "Error message",
  "details": ["Additional error details (optional)"]
}
```

### Common HTTP Status Codes

| Status | Meaning | Common Causes |
|--------|---------|---------------|
| `400` | Bad Request | Invalid request data, validation errors |
| `401` | Unauthorized | Missing or invalid API key |
| `402` | Payment Required | Insufficient credits |
| `404` | Not Found | Invalid state/district/SRO codes |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server-side error |

### Example Error Responses

**Invalid State (400):**
```json
{
  "error": "Invalid state: Delhii. Available states: Assam, Delhi, Haryana, ..."
}
```

**Insufficient Credits (402):**
```json
{
  "error": "Insufficient credits. Required: ₹1125, Available: ₹500",
  "details": {
    "required": 1125,
    "available": 500,
    "shortfall": 625
  }
}
```

**Invalid API Key (401):**
```json
{
  "error": "Invalid API key. Please check your credentials."
}
```

## 💳 Rate Limits & Credits

### Credits System

- **All stamp orders deduct credits** from your workspace balance
- **Credits = Indian Rupees** (₹1 credit = ₹1)
- **Order cost includes**: Stamp amount + convenience fee + delivery charges + taxes
- **Auto-applicable coupons** are applied when available

### Adding Credits

1. Go to your **eSahayak Dashboard** → **Settings**
2. Click **Add Credits** in the API Access section
3. Purchase credits using UPI/Card/Net Banking
4. **Minimum purchase**: ₹250

### Rate Limits

- **1000 requests per hour** per API key
- **50 stamp orders per hour** per workspace
- **Contact support** for higher limits

## 📚 Examples

### Complete Integration Flow

```javascript
// 1. Get available states
const states = await fetch('https://esahayak.io/api/v1/data/states')
  .then(r => r.json());

// 2. Get districts for selected state
const districts = await fetch('https://esahayak.io/api/v1/data/districts?stateCode=AS')
  .then(r => r.json());

// 3. Get SROs for selected district (if required)
const sros = await fetch('https://esahayak.io/api/v1/data/sros?stateCode=AS&districtCode=AS-MO')
  .then(r => r.json());

// 4. Get available articles
const articles = await fetch('https://esahayak.io/api/v1/data/articles?stateCode=AS')
  .then(r => r.json());

// 5. Create stamp order
const order = await fetch('https://esahayak.io/api/v1/stamp-order', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': 'YOUR_API_KEY_HERE'
  },
  body: JSON.stringify({
    state: {
      name: "Assam",
      district: "AS-MO",
      sro: "AS-MRG"
    },
    stampAmount: 1000,
    articleCode: "AS-RG-4",
    // ... other required fields
  })
}).then(r => r.json());

console.log('Order created:', order.orderId);
```

### Python Example

```python
import requests

# Configuration
BASE_URL = "https://esahayak.io/api/v1"
API_KEY = "YOUR_API_KEY_HERE"

headers = {
    "Content-Type": "application/json",
    "x-api-key": API_KEY
}

# Get states
states = requests.get(f"{BASE_URL}/data/states").json()
print(f"Available states: {len(states['data'])}")

# Create stamp order
order_data = {
    "state": {
        "name": "Assam",
        "district": "AS-MO",
        "sro": "AS-MRG"
    },
    "stampAmount": 1000,
    "articleCode": "AS-RG-4",
    "firstParty": {"name": "John Doe", "phone": "9876543210"},
    "stampDutyPayer": {
        "party": "first",
        "gender": "male",
        "panCardFileUrl": "https://example.com/pan.jpg"
    },
    "purpose": "Affidavit for property verification"
}

response = requests.post(f"{BASE_URL}/stamp-order",
                        json=order_data, headers=headers)

if response.status_code == 201:
    order = response.json()
    print(f"Order created: {order['orderId']}")
    print(f"Credits remaining: ₹{order['data']['credits']['remainingBalance']}")
else:
    print(f"Error: {response.json()['error']}")
```

## 🆘 Support

- **Documentation Issues**: Open an issue on GitHub
- **API Support**: Contact support@esahayak.io
- **Status Page**: [https://status.esahayak.io](https://status.esahayak.io)

---

**Happy Building! 🚀**

For more examples and advanced usage, check out our [GitHub repository](https://github.com/esahayak/api-examples).
