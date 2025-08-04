# ⚡ Quick Start Guide

Get up and running with the eSahayak API in 5 minutes!

## 1. 🔑 Get Your API Key

1. **Sign up** at [https://esahayak.io](https://esahayak.io)
2. **Complete your profile** and workspace setup
3. **Go to Settings** → **API Access**
4. **Click "Generate API Key"**
5. **Copy your API key** - you'll need this for all requests

## 2. 💳 Add Credits

Before creating stamp orders, you need credits in your workspace:

1. **In Settings** → **API Access** → **Credits section**
2. **Click "Add Credits"**
3. **Purchase minimum ₹250** (recommended: ₹1000+ for testing)
4. **Complete payment** via UPI/Card/Net Banking

## 3. 🧪 Test Your First API Call

Let's start with getting available states (no auth required):

```bash
curl https://esahayak.io/api/v1/data/states
```

You should see a JSON response with all available states.

## 4. 🎯 Create Your First Stamp Order

Here's a minimal example for creating an Affidavit stamp in Assam:

```bash
curl -X POST https://esahayak.io/api/v1/stamp-order \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -d '{
    "state": {
      "name": "Assam",
      "district": "AS-MO",
      "sro": "AS-MRG"
    },
    "stampAmount": 100,
    "cessAmount": 0,
    "firstParty": {
      "name": "John Doe",
      "phone": "9876543210"
    },
    "stampDutyPayer": {
      "party": "first",
      "gender": "male",
      "panCardFileUrl": "https://example.com/pan-card.jpg"
    },
    "articleCode": "AS-RG-4",
    "purpose": "Affidavit for property verification"
  }'
```

## 5. ✅ Success Response

If everything works correctly, you'll get:

```json
{
  "success": true,
  "orderId": "order_12345",
  "stampRequestId": "stamp_67890",
  "message": "Stamp order created successfully",
  "data": {
    "credits": {
      "previousBalance": 1000,
      "amountDeducted": 115,
      "remainingBalance": 885
    }
  }
}
```

🎉 **Congratulations!** You've successfully created your first stamp order.

## 🔄 Next Steps

1. **Read the [Full Documentation](README.md)** for detailed API reference
2. **Explore [Data APIs](data-apis.md)** to understand how to get required codes
3. **Check [Examples](examples.md)** for more complex use cases
4. **Set up [Webhooks](webhooks.md)** to get notified when stamps are ready

## 🚨 Common Issues

### ❌ "Invalid API key"
- **Solution**: Double-check your API key in workspace settings
- **Make sure**: You're using the correct header name: `x-api-key`

### ❌ "Insufficient credits"
- **Solution**: Add more credits to your workspace
- **Note**: Each order costs stamp amount + fees (usually 10-15% additional)

### ❌ "Invalid state/district/article code"
- **Solution**: Use the [Data APIs](data-apis.md) to get valid codes
- **Don't hardcode**: Always fetch current data from our APIs

### ❌ "Validation errors"
- **Solution**: Check the [Request Schema](README.md#create-stamp-order) carefully
- **Required fields**: Make sure all required fields are provided

## 📞 Need Help?

- **Documentation**: [Full API Docs](README.md)
- **Email**: [https://esahayak.io/contact-us](https://esahayak.io/contact-us)

Happy coding! 🚀
