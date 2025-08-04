# 🚀 eSahayak API Documentation

Welcome to the comprehensive eSahayak API documentation! Our API allows you to integrate stamp duty services seamlessly into your application.

## 📖 Documentation Sections

### 🔥 [Quick Start Guide](quickstart.md)
Get up and running in 5 minutes with step-by-step instructions.

### 📚 [Complete API Reference](README.md)
Detailed documentation for all endpoints, authentication, and error handling.

### 📊 [Data APIs Guide](data-apis.md)
Reference data endpoints for states, districts, SROs, and articles.

### 💻 [Code Examples](examples.md)
Ready-to-use examples in JavaScript, Python, PHP, Java, and more.

### 🔔 [Webhooks Documentation](webhooks.md)
Real-time notifications for order updates and status changes.

---

## 🌟 What You Can Build

### 🏢 **Business Applications**
- **Legal document platforms** - Automated stamp duty for contracts
- **Property portals** - Stamp duty for property transactions
- **HR platforms** - Affidavits and agreements for employees
- **Fintech apps** - Stamp duty for loan agreements

### 🛠️ **Developer Tools**
- **Document workflows** - Integrate stamp duty into existing flows
- **Compliance tools** - Ensure legal document validity
- **Automation platforms** - Bulk stamp order processing
- **API gateways** - White-label stamp duty services

---

## ⚡ Quick Overview

### 🔐 Authentication
```http
x-api-key: YOUR_API_KEY_HERE
```

### 🌍 Base URL
```
https://esahayak.io/api/v1
```

### 📊 Available APIs

| Category | Endpoints | Purpose |
|----------|-----------|---------|
| **Data APIs** | `/data/states`, `/data/districts`, `/data/sros`, `/data/articles` | Reference data (no auth) |
| **Stamp Orders** | `/stamp-order` | Create stamp orders (requires auth) |

### 💰 Pricing Model
- **Credits-based** - ₹1 credit = ₹1
- **Pay-as-you-use** - Only pay for successful orders
- **No monthly fees** - Credits never expire
- **Transparent pricing** - Cost = stamp amount + fees

---

## 🚦 Getting Started Checklist

- [ ] **Create eSahayak account** at [esahayak.io](https://esahayak.io)
- [ ] **Generate API key** in Settings → API Access
- [ ] **Add credits** to your workspace (minimum ₹250)
- [ ] **Test with data APIs** (no auth required)
- [ ] **Create your first stamp order**
- [ ] **Set up webhooks** for real-time updates

---

## 📋 Common Use Cases

### 1. **Simple Affidavit Creation**
```javascript
const order = await api.createStampOrder({
  state: { name: "Delhi" },
  stampAmount: 100,
  articleCode: "DL-RG-4",
  firstParty: { name: "John Doe", phone: "9876543210" },
  stampDutyPayer: { party: "first", gender: "male", panCardFileUrl: "..." },
  purpose: "Affidavit for address proof"
});
```

### 2. **Property Agreement**
```javascript
const order = await api.createStampOrder({
  state: { name: "Maharashtra", district: "MH-MU", sro: "MH-MU-01" },
  stampAmount: 5000,
  cessAmount: 100,
  articleCode: "MH-RG-23",
  propertyArea: { unit: "SQUARE_FEET", value: 1000 },
  // ... other required fields
});
```

### 3. **Bulk Processing**
```javascript
const orders = [
  { /* order 1 */ },
  { /* order 2 */ },
  { /* order 3 */ }
];

const results = await Promise.allSettled(
  orders.map(order => api.createStampOrder(order))
);
```

---

## 🔍 API Features

### ✅ **Smart Validation**
- **Real-time validation** of state/district/article combinations
- **Amount validation** based on state and article rules
- **Business logic enforcement** for compliance

### ✅ **Automatic Cost Calculation**
- **Dynamic pricing** based on state rules
- **Auto-applicable coupons** for cost optimization
- **Transparent breakdown** of all charges

### ✅ **Comprehensive Error Handling**
- **Detailed error messages** with actionable guidance
- **Validation errors** with specific field information
- **Business logic errors** with resolution steps

### ✅ **Real-time Updates**
- **Webhook notifications** for order status changes
- **Delivery tracking** for physical stamps
- **Status monitoring** throughout the process

---

## 🛡️ Security & Reliability

### 🔐 **Security**
- **API key authentication** for all sensitive operations
- **HTTPS encryption** for all communications
- **Input validation** to prevent malicious requests
- **Rate limiting** to prevent abuse

### 🔄 **Reliability**
- **99.9% uptime SLA** for critical operations
- **Automatic retries** for transient failures
- **Graceful degradation** during maintenance
- **Comprehensive monitoring** and alerting

### 📊 **Monitoring**
- **Real-time status page** at [status.esahayak.io](https://status.esahayak.io)
- **API response time monitoring**
- **Error rate tracking**
- **Capacity planning** for peak loads

---

## 🎯 Integration Patterns

### 1. **Synchronous Integration**
Best for: Real-time applications, simple workflows
```javascript
// User submits form → API call → Immediate response
const order = await createStampOrder(formData);
showSuccess(`Order created: ${order.orderId}`);
```

### 2. **Asynchronous Integration**
Best for: Complex workflows, bulk processing
```javascript
// Queue order → Background processing → Webhook notification
await queueStampOrder(orderData);
// Process webhook to update UI
```

### 3. **Batch Processing**
Best for: High-volume applications, scheduled processing
```javascript
// Process orders in batches with rate limiting
const results = await processBatchOrders(orders, { batchSize: 10 });
```

---

## 📈 Performance Guidelines

### ⚡ **Rate Limits**
- **1000 requests/hour** per API key
- **50 stamp orders/hour** per workspace
- **Contact support** for higher limits

### 🎯 **Best Practices**
- **Cache reference data** (states, districts, articles)
- **Implement retry logic** with exponential backoff
- **Use bulk operations** for multiple orders
- **Monitor webhook delivery** success rates

### 🔧 **Optimization Tips**
- **Parallel API calls** for independent operations
- **Client-side validation** before API calls
- **Efficient error handling** with proper logging
- **Connection pooling** for high-volume scenarios

---

## 🆘 Support & Resources

### 📧 **Contact Support**
- **API Issues**: support@esahayak.io
- **Business Inquiries**: sales@esahayak.io
- **Technical Support**: Available 9 AM - 6 PM IST

### 📚 **Additional Resources**
- **GitHub Examples**: [github.com/esahayak/api-examples](https://github.com/esahayak/api-examples)
- **Postman Collection**: [Download Collection](https://api.esahayak.io/postman)
- **OpenAPI Spec**: [Download Spec](https://api.esahayak.io/openapi.json)
- **Status Page**: [status.esahayak.io](https://status.esahayak.io)

### 🤝 **Community**
- **Developer Forum**: [forum.esahayak.io](https://forum.esahayak.io)
- **Discord Channel**: [discord.gg/esahayak](https://discord.gg/esahayak)
- **Stack Overflow**: Tag your questions with `esahayak-api`

---

## 🔄 Changelog & Updates

### **v1.0.0** - January 2024
- ✅ Initial API release
- ✅ Data APIs for reference information
- ✅ Stamp order creation with validation
- ✅ Webhook notifications
- ✅ Multi-language examples

### **Coming Soon**
- 🔜 Order status tracking API
- 🔜 Bulk order operations
- 🔜 Advanced webhook filtering
- 🔜 GraphQL endpoint

---

## 💡 Feature Requests

Have ideas for improving the API? We'd love to hear from you!

- **Submit issues**: [GitHub Issues](https://github.com/esahayak/api-docs/issues)
- **Feature requests**: support@esahayak.io
- **API improvements**: Include your use case and expected behavior

---

**Ready to get started? 🚀**

Choose your path:
- **New to APIs?** → [Quick Start Guide](quickstart.md)
- **Want details?** → [Complete API Reference](README.md)
- **Need examples?** → [Code Examples](examples.md)
- **Building integrations?** → [Webhooks Guide](webhooks.md)

---

*Happy building! The eSahayak team is here to support your integration journey.*
