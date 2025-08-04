# 🔔 Webhooks

Get real-time notifications when stamp orders are processed, completed, or encounter issues.

## 📋 Table of Contents

- [Overview](#overview)
- [Setup](#setup)
- [Event Types](#event-types)
- [Payload Format](#payload-format)
- [Security](#security)
- [Testing](#testing)
- [Examples](#examples)

---

## 🌟 Overview

Webhooks allow your application to receive real-time notifications about stamp order status changes. Instead of polling our API for updates, we'll send HTTP POST requests to your specified endpoint whenever important events occur.

### Benefits
- **Real-time updates** - Get notified immediately when orders are processed
- **Reduced API calls** - No need to poll for status updates
- **Better user experience** - Notify users instantly when stamps are ready
- **Automation** - Trigger downstream processes automatically

---

## ⚙️ Setup

### 1. Configure Webhook URL

In your eSahayak dashboard:

1. Go to **Settings** → **API Access**
2. Find **Webhook Configuration** section
3. Enter your webhook endpoint URL
4. Click **Save Webhook**

```
https://your-domain.com/webhooks/esahayak
```

### 2. Endpoint Requirements

Your webhook endpoint must:
- **Accept POST requests** with JSON payload
- **Respond with 2xx status code** (200, 201, 204) within 10 seconds
- **Use HTTPS** (HTTP not supported)
- **Be publicly accessible** (no authentication required for the endpoint itself)

### 3. Example Endpoint Implementation

```javascript
// Express.js example
app.post('/webhooks/esahayak', express.json(), (req, res) => {
  const event = req.body;

  console.log('Received webhook:', event.type);

  try {
    // Process the webhook event
    handleWebhookEvent(event);

    // Return success response
    res.status(200).json({ received: true });
  } catch (error) {
    console.error('Webhook processing failed:', error);
    res.status(500).json({ error: 'Processing failed' });
  }
});
```

---

## 📨 Event Types

### `stamp.uploaded`

Triggered when a stamp is successfully generated and uploaded.

**When**: Stamp is ready for download/delivery
**Action**: Notify user, trigger delivery process

### `stamp.failed`

Triggered when stamp generation fails due to technical issues.

**When**: Government portal issues, technical errors
**Action**: Retry logic, notify user of delay

### `order.delivered`

Triggered when physical stamp delivery is completed (if delivery was requested).

**When**: Delivery partner confirms successful delivery
**Action**: Update order status, collect feedback

### `order.cancelled`

Triggered when an order is cancelled (manual intervention or technical issues).

**When**: Order cannot be processed, refund initiated
**Action**: Process refund, notify user

---

## 📦 Payload Format

All webhook payloads follow this structure:

```json
{
  "id": "webhook_unique_id",
  "type": "stamp.uploaded",
  "created_at": "2024-01-15T10:30:00Z",
  "data": {
    "order": {
      "id": "order_12345",
      "status": "COMPLETED",
      "created_at": "2024-01-15T10:00:00Z"
    },
    "stamp_request": {
      "id": "stamp_67890",
      "state": "Assam",
      "amount": 1000,
      "article_code": "AS-RG-4",
      "provider": "SHCIL"
    }
  }
}
```

### Common Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique webhook event ID |
| `type` | string | Event type (see above) |
| `created_at` | string | ISO 8601 timestamp |
| `data` | object | Event-specific payload |

---

## 🔐 Security

### Request Headers

We include these headers with every webhook request:

```http
POST /webhooks/esahayak HTTP/1.1
Host: your-domain.com
Content-Type: application/json
User-Agent: eSahayak-Webhooks/1.0
X-eSahayak-Event: stamp.uploaded
X-eSahayak-Delivery: webhook_12345
```

### Signature Verification

**Coming Soon**: We'll add HMAC signature verification for enhanced security.

### IP Allowlisting

Webhook requests come from these IP ranges:
- `52.66.123.0/24` (Primary)
- `13.127.45.0/24` (Backup)

### Best Practices

1. **Validate payload structure** before processing
2. **Store webhook events** for debugging
3. **Implement idempotency** - handle duplicate events gracefully
4. **Use HTTPS** - never HTTP for webhooks
5. **Monitor webhook endpoints** - ensure they're always available

---

## 🧪 Testing

### 1. Test Endpoint

Use webhook.site for testing:

```bash
# Generate a test URL at https://webhook.site
# Use that URL in your webhook configuration
# Create a test order and watch for events
```

### 2. Local Development

Use ngrok to expose local development server:

```bash
# Install ngrok
npm install -g ngrok

# Start your local server
node server.js # Running on port 3000

# Expose via ngrok (in another terminal)
ngrok http 3000

# Use the ngrok URL in webhook configuration
# https://abc123.ngrok.io/webhooks/esahayak
```

### 3. Webhook Testing Tool

```javascript
// Simple webhook testing server
const express = require('express');
const app = express();

app.use(express.json());

app.post('/webhooks/esahayak', (req, res) => {
  console.log('\n🔔 Webhook Received:');
  console.log('Type:', req.body.type);
  console.log('Order ID:', req.body.data?.order?.id);
  console.log('Timestamp:', req.body.created_at);
  console.log('Full payload:', JSON.stringify(req.body, null, 2));

  res.status(200).json({
    received: true,
    timestamp: new Date().toISOString()
  });
});

app.listen(3000, () => {
  console.log('🎣 Webhook server listening on port 3000');
  console.log('Use ngrok to expose: ngrok http 3000');
});
```

---

## 💻 Examples

### Complete Webhook Handler

```javascript
const crypto = require('crypto');
const express = require('express');
const app = express();

// Webhook event handlers
const eventHandlers = {
  'stamp.uploaded': handleStampUploaded,
  'stamp.failed': handleStampFailed,
  'order.delivered': handleOrderDelivered,
  'order.cancelled': handleOrderCancelled
};

app.post('/webhooks/esahayak', express.raw({type: 'application/json'}), (req, res) => {
  try {
    // Parse the payload
    const payload = JSON.parse(req.body);
    const eventType = payload.type;

    console.log(`📥 Received webhook: ${eventType}`);

    // Validate event structure
    if (!isValidWebhookPayload(payload)) {
      console.error('❌ Invalid webhook payload');
      return res.status(400).json({ error: 'Invalid payload' });
    }

    // Handle the event
    const handler = eventHandlers[eventType];
    if (handler) {
      await handler(payload.data);
      console.log(`✅ Processed ${eventType} successfully`);
    } else {
      console.log(`⚠️ Unhandled event type: ${eventType}`);
    }

    // Always respond with success
    res.status(200).json({
      received: true,
      event_id: payload.id,
      processed_at: new Date().toISOString()
    });

  } catch (error) {
    console.error('❌ Webhook processing failed:', error);
    res.status(500).json({ error: 'Processing failed' });
  }
});

// Event handler functions
async function handleStampUploaded(data) {
  const orderId = data.order.id;
  const stampRequestId = data.stamp_request.id;

  // Update database
  await updateOrderStatus(orderId, 'COMPLETED');

  // Notify user via email/SMS
  await notifyUser(orderId, 'Your stamp is ready for download');

  // Trigger any downstream processes
  await triggerDeliveryProcess(orderId);

  console.log(`🎉 Stamp uploaded for order ${orderId}`);
}

async function handleStampFailed(data) {
  const orderId = data.order.id;
  const reason = data.error?.message || 'Technical issue';

  // Update database
  await updateOrderStatus(orderId, 'FAILED');

  // Notify user
  await notifyUser(orderId, `Stamp generation failed: ${reason}`);

  // Log for investigation
  await logFailure(orderId, reason);

  console.log(`❌ Stamp failed for order ${orderId}: ${reason}`);
}

async function handleOrderDelivered(data) {
  const orderId = data.order.id;
  const deliveryDetails = data.delivery;

  // Update delivery status
  await updateDeliveryStatus(orderId, 'DELIVERED', deliveryDetails);

  // Send completion notification
  await notifyUser(orderId, 'Your stamp has been delivered successfully');

  // Request feedback
  await sendFeedbackRequest(orderId);

  console.log(`📦 Order ${orderId} delivered successfully`);
}

async function handleOrderCancelled(data) {
  const orderId = data.order.id;
  const reason = data.cancellation?.reason || 'Unknown';

  // Update order status
  await updateOrderStatus(orderId, 'CANCELLED');

  // Process refund if applicable
  await processRefund(orderId);

  // Notify user
  await notifyUser(orderId, `Order cancelled: ${reason}. Refund processed.`);

  console.log(`🚫 Order ${orderId} cancelled: ${reason}`);
}

// Utility functions
function isValidWebhookPayload(payload) {
  return payload.id &&
         payload.type &&
         payload.created_at &&
         payload.data;
}

async function updateOrderStatus(orderId, status) {
  // Your database update logic
  console.log(`📝 Updated order ${orderId} status to ${status}`);
}

async function notifyUser(orderId, message) {
  // Your notification logic (email, SMS, push notification)
  console.log(`📧 Notified user for order ${orderId}: ${message}`);
}

app.listen(3000, () => {
  console.log('🎣 Webhook server running on port 3000');
});
```

### Database Integration (MongoDB)

```javascript
const mongoose = require('mongoose');

// Order schema
const orderSchema = new mongoose.Schema({
  esahayakOrderId: String,
  status: String,
  webhookEvents: [{
    type: String,
    data: Object,
    receivedAt: Date
  }],
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

const Order = mongoose.model('Order', orderSchema);

// Webhook handler with database persistence
async function handleStampUploaded(data) {
  const orderId = data.order.id;

  // Find and update the order
  const order = await Order.findOne({ esahayakOrderId: orderId });
  if (order) {
    order.status = 'COMPLETED';
    order.updatedAt = new Date();
    order.webhookEvents.push({
      type: 'stamp.uploaded',
      data: data,
      receivedAt: new Date()
    });

    await order.save();
    console.log(`✅ Updated order ${orderId} in database`);

    // Trigger additional business logic
    await triggerOrderCompletionWorkflow(order);
  } else {
    console.error(`❌ Order ${orderId} not found in database`);
  }
}
```

### Real-time Updates (WebSocket)

```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

// Store client connections by user ID
const userConnections = new Map();

wss.on('connection', (ws, req) => {
  const userId = extractUserIdFromRequest(req);
  userConnections.set(userId, ws);

  ws.on('close', () => {
    userConnections.delete(userId);
  });
});

// Send real-time updates to users
async function notifyUserRealtime(orderId, message, eventType) {
  const order = await Order.findOne({ esahayakOrderId: orderId });
  if (order && order.userId) {
    const userWs = userConnections.get(order.userId);
    if (userWs && userWs.readyState === WebSocket.OPEN) {
      userWs.send(JSON.stringify({
        type: 'order_update',
        orderId: orderId,
        status: eventType,
        message: message,
        timestamp: new Date().toISOString()
      }));
    }
  }
}

// Enhanced webhook handler with real-time updates
async function handleStampUploaded(data) {
  const orderId = data.order.id;

  // Update database
  await updateOrderStatus(orderId, 'COMPLETED');

  // Send real-time update
  await notifyUserRealtime(orderId, 'Your stamp is ready!', 'completed');

  // Send email notification
  await sendEmailNotification(orderId);
}
```

---

## 🔍 Debugging

### Common Issues

#### ❌ Webhook not received
- **Check URL**: Ensure webhook URL is correct and publicly accessible
- **Check firewall**: Ensure your server accepts incoming connections
- **Check logs**: Look for any server errors or timeouts
- **Test manually**: Use curl to test your endpoint

#### ❌ Webhook timeout
- **Response time**: Ensure your endpoint responds within 10 seconds
- **Async processing**: Move heavy processing to background jobs
- **Database connections**: Ensure database isn't blocking

#### ❌ Duplicate events
- **Implement idempotency**: Use webhook ID to detect duplicates
- **Database constraints**: Use unique constraints where appropriate

### Debugging Tools

```javascript
// Webhook logger middleware
app.use('/webhooks', (req, res, next) => {
  console.log(`📝 Webhook ${req.method} ${req.path}`);
  console.log('Headers:', req.headers);
  console.log('Body:', req.body);

  // Continue to actual handler
  next();
});

// Webhook response logger
app.use('/webhooks', (req, res, next) => {
  const originalSend = res.send;
  res.send = function(body) {
    console.log(`📤 Response ${res.statusCode}: ${body}`);
    return originalSend.call(this, body);
  };
  next();
});
```

### Health Check Endpoint

```javascript
app.get('/webhooks/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    server: 'webhook-handler',
    version: '1.0.0'
  });
});
```

---

## 📈 Monitoring

### Webhook Metrics

Track these metrics for webhook reliability:

- **Delivery success rate** - % of webhooks delivered successfully
- **Response time** - How quickly your endpoint responds
- **Error rate** - % of webhooks that result in errors
- **Retry attempts** - Number of retries needed

### Example Monitoring

```javascript
const metrics = {
  total: 0,
  success: 0,
  errors: 0,
  averageResponseTime: 0
};

app.post('/webhooks/esahayak', async (req, res) => {
  const startTime = Date.now();
  metrics.total++;

  try {
    // Process webhook
    await processWebhook(req.body);

    metrics.success++;
    res.status(200).json({ received: true });
  } catch (error) {
    metrics.errors++;
    res.status(500).json({ error: 'Processing failed' });
  } finally {
    const responseTime = Date.now() - startTime;
    metrics.averageResponseTime =
      (metrics.averageResponseTime + responseTime) / 2;
  }
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.json({
    ...metrics,
    successRate: (metrics.success / metrics.total * 100).toFixed(2) + '%',
    errorRate: (metrics.errors / metrics.total * 100).toFixed(2) + '%'
  });
});
```

---

## 🆘 Support

- **Webhook Issues**: support@esahayak.io
- **Documentation**: [Full API Docs](README.md)
- **Status Page**: [status.esahayak.io](https://status.esahayak.io)

For webhook-specific support, include:
1. Webhook URL being used
2. Expected vs actual behavior
3. Relevant server logs
4. Timestamp of the issue
