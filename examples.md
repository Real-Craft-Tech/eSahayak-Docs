# 💻 Code Examples

Comprehensive examples for integrating the eSahayak API in different programming languages.

## 📋 Table of Contents

- [JavaScript/Node.js](#javascriptnodejs)
- [Python](#python)
- [PHP](#php)
- [Java](#java)
- [cURL](#curl)
- [Common Patterns](#common-patterns)

---

## 🟨 JavaScript/Node.js

### Basic Setup

```javascript
const API_BASE = 'https://esahayak.io/api/v1';
const API_KEY = process.env.ESAHAYAK_API_KEY; // Store in environment variable

class eSahayakAPI {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseURL = API_BASE;
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.apiKey && { 'x-api-key': this.apiKey }),
      },
      ...options,
    };

    const response = await fetch(url, config);
    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || `HTTP ${response.status}`);
    }

    return data;
  }

  // Data APIs (no auth required)
  async getStates() {
    return this.request('/data/states');
  }

  async getDistricts(stateCode) {
    return this.request(`/data/districts?stateCode=${stateCode}`);
  }

  async getSROs(stateCode, districtCode) {
    return this.request(`/data/sros?stateCode=${stateCode}&districtCode=${districtCode}`);
  }

  async getArticles(stateCode) {
    return this.request(`/data/articles?stateCode=${stateCode}`);
  }

  // Stamp Order API (auth required)
  async createStampOrder(orderData) {
    return this.request('/stamp-order', {
      method: 'POST',
      body: JSON.stringify(orderData),
    });
  }
}
```

### Complete Example: Create an Affidavit

```javascript
async function createAffidavitOrder() {
  const api = new eSahayakAPI(API_KEY);

  try {
    // 1. Get states and find Assam
    const states = await api.getStates();
    const assam = states.data.find(state => state.name === 'Assam');

    if (!assam) throw new Error('Assam not available');

    // 2. Get districts for Assam
    const districts = await api.getDistricts(assam.code);
    const morigaon = districts.data.find(d => d.name === 'Morigaon');

    // 3. Get SROs for Morigaon (if required)
    let sroCode = null;
    if (assam.sroRequired && morigaon) {
      const sros = await api.getSROs(assam.code, morigaon.code);
      sroCode = sros.data[0]?.code;
    }

    // 4. Get articles and find Affidavit
    const articles = await api.getArticles(assam.code);
    const affidavit = articles.data.find(a => a.description.includes('Affidavit'));

    // 5. Create the order
    const orderData = {
      state: {
        name: assam.name,
        ...(morigaon && { district: morigaon.code }),
        ...(sroCode && { sro: sroCode }),
      },
      stampAmount: Math.max(affidavit.minimumAmount, 100), // Use minimum or ₹100
      cessAmount: assam.cessRequired ? 10 : 0,
      firstParty: {
        name: 'John Doe',
        phone: '9876543210',
      },
      stampDutyPayer: {
        party: 'first',
        gender: 'male',
        panCardFileUrl: 'https://example.com/pan-card.jpg',
      },
      articleCode: affidavit.code,
      purpose: 'Affidavit for property verification documents',
      deliveryAddresses: [{
        name: 'John Doe',
        address: '123 Main Street, Guwahati',
        pincode: '781001',
        phone: '9876543210',
        email: 'john@example.com',
      }],
    };

    const result = await api.createStampOrder(orderData);

    console.log('✅ Order created successfully!');
    console.log(`Order ID: ${result.orderId}`);
    console.log(`Total Cost: ₹${result.data.pricing.discountedTotal}`);
    console.log(`Credits Remaining: ₹${result.data.credits.remainingBalance}`);

    return result;

  } catch (error) {
    console.error('❌ Error creating order:', error.message);
    throw error;
  }
}

// Usage
createAffidavitOrder()
  .then(order => {
    // Handle success
    console.log('Order created:', order.orderId);
  })
  .catch(error => {
    // Handle error
    console.error('Failed to create order:', error.message);
  });
```

### React Hook Example

```javascript
import { useState, useEffect } from 'react';

function useeSahayakAPI(apiKey) {
  const [states, setStates] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const api = new eSahayakAPI(apiKey);

  useEffect(() => {
    loadStates();
  }, []);

  const loadStates = async () => {
    try {
      setLoading(true);
      const result = await api.getStates();
      setStates(result.data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const createOrder = async (orderData) => {
    try {
      setLoading(true);
      const result = await api.createStampOrder(orderData);
      setError(null);
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { states, loading, error, createOrder, api };
}

// Component usage
function StampOrderForm() {
  const { states, loading, error, createOrder } = useeSahayakAPI(API_KEY);

  const handleSubmit = async (formData) => {
    try {
      const order = await createOrder(formData);
      alert(`Order created: ${order.orderId}`);
    } catch (error) {
      alert(`Error: ${error.message}`);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <form onSubmit={handleSubmit}>
      {/* Your form fields */}
    </form>
  );
}
```

---

## 🐍 Python

### Basic Setup

```python
import requests
import os
from typing import Dict, List, Optional

class eSahayakAPI:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://esahayak.io/api/v1"
        self.session = requests.Session()

    def _request(self, endpoint: str, method: str = "GET", data: Dict = None) -> Dict:
        url = f"{self.base_url}{endpoint}"
        headers = {"Content-Type": "application/json"}

        if self.api_key:
            headers["x-api-key"] = self.api_key

        response = self.session.request(
            method=method,
            url=url,
            json=data,
            headers=headers
        )

        result = response.json()

        if not response.ok:
            raise Exception(f"API Error: {result.get('error', 'Unknown error')}")

        return result

    # Data APIs
    def get_states(self) -> Dict:
        return self._request("/data/states")

    def get_districts(self, state_code: str) -> Dict:
        return self._request(f"/data/districts?stateCode={state_code}")

    def get_sros(self, state_code: str, district_code: str) -> Dict:
        return self._request(f"/data/sros?stateCode={state_code}&districtCode={district_code}")

    def get_articles(self, state_code: str) -> Dict:
        return self._request(f"/data/articles?stateCode={state_code}")

    # Stamp Order API
    def create_stamp_order(self, order_data: Dict) -> Dict:
        return self._request("/stamp-order", method="POST", data=order_data)
```

### Complete Example: Batch Processing

```python
import json
import time
from datetime import datetime

class StampOrderProcessor:
    def __init__(self, api_key: str):
        self.api = eSahayakAPI(api_key)
        self.cache = {}

    def load_reference_data(self):
        """Load and cache all reference data"""
        print("Loading reference data...")

        # Load states
        states = self.api.get_states()
        self.cache['states'] = {state['name']: state for state in states['data']}

        # Load districts and articles for each state
        for state_name, state_info in self.cache['states'].items():
            state_code = state_info['code']

            # Load districts
            if state_info.get('sroRequired'):
                districts = self.api.get_districts(state_code)
                self.cache[f'districts_{state_code}'] = districts['data']

            # Load articles
            articles = self.api.get_articles(state_code)
            self.cache[f'articles_{state_code}'] = {
                article['code']: article for article in articles['data']
            }

        print(f"✅ Loaded data for {len(self.cache['states'])} states")

    def create_order_from_template(self, template: Dict) -> Dict:
        """Create order from a template with validation"""
        state_name = template['state']['name']
        state_info = self.cache['states'].get(state_name)

        if not state_info:
            raise ValueError(f"State '{state_name}' not supported")

        # Validate article code
        article_code = template['articleCode']
        articles = self.cache[f'articles_{state_info["code"]}']
        article = articles.get(article_code)

        if not article:
            raise ValueError(f"Article '{article_code}' not available for {state_name}")

        # Validate stamp amount
        stamp_amount = template['stampAmount']
        if stamp_amount < article['minimumAmount']:
            raise ValueError(f"Minimum amount for {article_code} is ₹{article['minimumAmount']}")

        if article['allowedAmounts'] and stamp_amount not in article['allowedAmounts']:
            raise ValueError(f"Allowed amounts: {article['allowedAmounts']}")

        # Create the order
        return self.api.create_stamp_order(template)

    def process_batch(self, orders: List[Dict]) -> List[Dict]:
        """Process multiple orders with error handling"""
        results = []

        for i, order_template in enumerate(orders):
            try:
                print(f"Processing order {i+1}/{len(orders)}...")
                result = self.create_order_from_template(order_template)

                results.append({
                    'status': 'success',
                    'order_id': result['orderId'],
                    'cost': result['data']['pricing']['discountedTotal'],
                    'credits_remaining': result['data']['credits']['remainingBalance']
                })

                print(f"✅ Order {i+1} created: {result['orderId']}")

                # Rate limiting
                time.sleep(1)

            except Exception as e:
                print(f"❌ Order {i+1} failed: {str(e)}")
                results.append({
                    'status': 'error',
                    'error': str(e),
                    'template': order_template
                })

        return results

# Usage example
def main():
    api_key = os.getenv('ESAHAYAK_API_KEY')
    processor = StampOrderProcessor(api_key)

    # Load reference data once
    processor.load_reference_data()

    # Define order templates
    orders = [
        {
            "state": {"name": "Assam", "district": "AS-MO", "sro": "AS-MRG"},
            "stampAmount": 100,
            "articleCode": "AS-RG-4",
            "firstParty": {"name": "John Doe", "phone": "9876543210"},
            "stampDutyPayer": {"party": "first", "gender": "male", "panCardFileUrl": "https://example.com/pan1.jpg"},
            "purpose": "Affidavit for property verification"
        },
        {
            "state": {"name": "Delhi"},
            "stampAmount": 50,
            "articleCode": "DL-RG-1",
            "firstParty": {"name": "Jane Smith", "phone": "9876543211"},
            "stampDutyPayer": {"party": "first", "gender": "female", "panCardFileUrl": "https://example.com/pan2.jpg"},
            "purpose": "Agreement for business partnership"
        }
    ]

    # Process batch
    results = processor.process_batch(orders)

    # Summary
    successful = len([r for r in results if r['status'] == 'success'])
    failed = len([r for r in results if r['status'] == 'error'])

    print(f"\n📊 Batch Summary:")
    print(f"✅ Successful: {successful}")
    print(f"❌ Failed: {failed}")

    # Save results
    with open(f'batch_results_{datetime.now().strftime("%Y%m%d_%H%M%S")}.json', 'w') as f:
        json.dump(results, f, indent=2)

if __name__ == "__main__":
    main()
```

---

## 🐘 PHP

### Basic Setup

```php
<?php

class eSahayakAPI {
    private $apiKey;
    private $baseUrl = 'https://esahayak.io/api/v1';
    private $httpClient;

    public function __construct($apiKey) {
        $this->apiKey = $apiKey;
        $this->httpClient = new \GuzzleHttp\Client();
    }

    private function request($endpoint, $method = 'GET', $data = null) {
        $url = $this->baseUrl . $endpoint;
        $headers = ['Content-Type' => 'application/json'];

        if ($this->apiKey) {
            $headers['x-api-key'] = $this->apiKey;
        }

        $options = [
            'headers' => $headers,
            'http_errors' => false
        ];

        if ($data) {
            $options['json'] = $data;
        }

        $response = $this->httpClient->request($method, $url, $options);
        $result = json_decode($response->getBody(), true);

        if ($response->getStatusCode() >= 400) {
            throw new Exception($result['error'] ?? 'Unknown error');
        }

        return $result;
    }

    // Data APIs
    public function getStates() {
        return $this->request('/data/states');
    }

    public function getDistricts($stateCode) {
        return $this->request("/data/districts?stateCode={$stateCode}");
    }

    public function getSROs($stateCode, $districtCode) {
        return $this->request("/data/sros?stateCode={$stateCode}&districtCode={$districtCode}");
    }

    public function getArticles($stateCode) {
        return $this->request("/data/articles?stateCode={$stateCode}");
    }

    // Stamp Order API
    public function createStampOrder($orderData) {
        return $this->request('/stamp-order', 'POST', $orderData);
    }
}

// Usage example
function createStampOrder() {
    $apiKey = $_ENV['ESAHAYAK_API_KEY'];
    $api = new eSahayakAPI($apiKey);

    try {
        // Get reference data
        $states = $api->getStates();
        $assam = array_filter($states['data'], function($state) {
            return $state['name'] === 'Assam';
        })[0];

        $articles = $api->getArticles($assam['code']);
        $affidavit = array_filter($articles['data'], function($article) {
            return strpos($article['description'], 'Affidavit') !== false;
        })[0];

        // Create order
        $orderData = [
            'state' => ['name' => 'Assam'],
            'stampAmount' => 100,
            'articleCode' => $affidavit['code'],
            'firstParty' => [
                'name' => 'John Doe',
                'phone' => '9876543210'
            ],
            'stampDutyPayer' => [
                'party' => 'first',
                'gender' => 'male',
                'panCardFileUrl' => 'https://example.com/pan.jpg'
            ],
            'purpose' => 'Affidavit for property verification'
        ];

        $result = $api->createStampOrder($orderData);

        echo "✅ Order created: " . $result['orderId'] . "\n";
        echo "💰 Cost: ₹" . $result['data']['pricing']['discountedTotal'] . "\n";

    } catch (Exception $e) {
        echo "❌ Error: " . $e->getMessage() . "\n";
    }
}

createStampOrder();
?>
```

---

## ☕ Java

### Basic Setup (using Spring Boot)

```java
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import org.springframework.stereotype.Service;

@Service
public class eSahayakAPIService {
    private final RestTemplate restTemplate;
    private final String apiKey;
    private final String baseUrl = "https://esahayak.io/api/v1";

    public eSahayakAPIService(RestTemplateBuilder builder,
                             @Value("${esahayak.api.key}") String apiKey) {
        this.restTemplate = builder.build();
        this.apiKey = apiKey;
    }

    private HttpHeaders createHeaders() {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        if (apiKey != null) {
            headers.set("x-api-key", apiKey);
        }
        return headers;
    }

    public StatesResponse getStates() {
        String url = baseUrl + "/data/states";
        HttpEntity<String> entity = new HttpEntity<>(createHeaders());

        ResponseEntity<StatesResponse> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, StatesResponse.class);

        return response.getBody();
    }

    public ArticlesResponse getArticles(String stateCode) {
        String url = baseUrl + "/data/articles?stateCode=" + stateCode;
        HttpEntity<String> entity = new HttpEntity<>(createHeaders());

        ResponseEntity<ArticlesResponse> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, ArticlesResponse.class);

        return response.getBody();
    }

    public OrderResponse createStampOrder(StampOrderRequest orderData) {
        String url = baseUrl + "/stamp-order";
        HttpEntity<StampOrderRequest> entity = new HttpEntity<>(orderData, createHeaders());

        ResponseEntity<OrderResponse> response = restTemplate.exchange(
            url, HttpMethod.POST, entity, OrderResponse.class);

        return response.getBody();
    }
}

// DTO Classes
public class StampOrderRequest {
    private StateDetails state;
    private Integer stampAmount;
    private Integer cessAmount;
    private PartyDetails firstParty;
    private StampDutyPayer stampDutyPayer;
    private String articleCode;
    private String purpose;

    // Getters and setters...
}

@RestController
@RequestMapping("/api/stamps")
public class StampController {

    @Autowired
    private eSahayakAPIService apiService;

    @PostMapping("/create-order")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody StampOrderRequest request) {
        try {
            OrderResponse response = apiService.createStampOrder(request);
            return ResponseEntity.ok(response);
        } catch (Exception e) {
            return ResponseEntity.badRequest()
                .body(new ErrorResponse("Failed to create order: " + e.getMessage()));
        }
    }
}
```

---

## 🌐 cURL

### Get States
```bash
curl -X GET "https://esahayak.io/api/v1/data/states"
```

### Get Articles for Assam
```bash
curl -X GET "https://esahayak.io/api/v1/data/articles?stateCode=AS"
```

### Create Stamp Order
```bash
curl -X POST "https://esahayak.io/api/v1/stamp-order" \
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

---

## 🔄 Common Patterns

### 1. Error Handling with Retries

```javascript
async function apiCallWithRetry(apiCall, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await apiCall();
    } catch (error) {
      if (attempt === maxRetries) throw error;

      // Exponential backoff
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));

      console.log(`Retry attempt ${attempt + 1}/${maxRetries}`);
    }
  }
}

// Usage
const result = await apiCallWithRetry(() =>
  api.createStampOrder(orderData)
);
```

### 2. Bulk Order Processing

```javascript
async function processBulkOrders(orders, batchSize = 5) {
  const results = [];

  for (let i = 0; i < orders.length; i += batchSize) {
    const batch = orders.slice(i, i + batchSize);

    const batchPromises = batch.map(async (order, index) => {
      try {
        // Add delay to avoid rate limiting
        await new Promise(resolve => setTimeout(resolve, index * 200));

        const result = await api.createStampOrder(order);
        return { success: true, orderId: result.orderId, order };
      } catch (error) {
        return { success: false, error: error.message, order };
      }
    });

    const batchResults = await Promise.all(batchPromises);
    results.push(...batchResults);

    console.log(`Processed batch ${Math.floor(i/batchSize) + 1}`);
  }

  return results;
}
```

### 3. Data Validation Helper

```javascript
class OrderValidator {
  constructor(api) {
    this.api = api;
    this.cache = new Map();
  }

  async validateOrder(orderData) {
    const errors = [];

    // Validate state
    const states = await this.getCachedStates();
    const state = states.find(s => s.name === orderData.state.name);
    if (!state) {
      errors.push(`Invalid state: ${orderData.state.name}`);
      return errors; // Can't continue without valid state
    }

    // Validate article
    const articles = await this.getCachedArticles(state.code);
    const article = articles.find(a => a.code === orderData.articleCode);
    if (!article) {
      errors.push(`Invalid article code: ${orderData.articleCode}`);
    }

    // Validate stamp amount
    if (article && orderData.stampAmount < article.minimumAmount) {
      errors.push(`Minimum amount for ${article.code} is ₹${article.minimumAmount}`);
    }

    // Validate state-specific requirements
    if (state.sroRequired && !orderData.state.sro) {
      errors.push('SRO is required for this state');
    }

    if (state.cessRequired && !orderData.cessAmount) {
      errors.push('Cess amount is required for this state');
    }

    return errors;
  }

  async getCachedStates() {
    if (!this.cache.has('states')) {
      const result = await this.api.getStates();
      this.cache.set('states', result.data);
    }
    return this.cache.get('states');
  }

  async getCachedArticles(stateCode) {
    const key = `articles_${stateCode}`;
    if (!this.cache.has(key)) {
      const result = await this.api.getArticles(stateCode);
      this.cache.set(key, result.data);
    }
    return this.cache.get(key);
  }
}
```

### 4. Real-time Order Status Tracking

```javascript
class OrderTracker {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.orders = new Map();
  }

  async trackOrder(orderId) {
    // Since we don't have a status endpoint yet,
    // this would work with webhooks or polling
    this.orders.set(orderId, {
      id: orderId,
      status: 'CREATED',
      createdAt: new Date(),
      updates: []
    });

    // In a real implementation, you'd set up webhook listeners
    // or implement polling logic here
  }

  onOrderUpdate(orderId, status, details) {
    const order = this.orders.get(orderId);
    if (order) {
      order.status = status;
      order.updates.push({
        status,
        details,
        timestamp: new Date()
      });

      // Trigger callbacks or events
      this.notifyOrderUpdate(order);
    }
  }

  notifyOrderUpdate(order) {
    console.log(`Order ${order.id} status: ${order.status}`);
    // Implement your notification logic here
  }
}
```

---

## 🚨 Production Best Practices

### 1. Environment Configuration
```javascript
const config = {
  apiKey: process.env.ESAHAYAK_API_KEY,
  environment: process.env.NODE_ENV,
  retryAttempts: process.env.ESAHAYAK_RETRY_ATTEMPTS || 3,
  timeoutMs: process.env.ESAHAYAK_TIMEOUT_MS || 30000,
  cacheEnabled: process.env.ESAHAYAK_CACHE_ENABLED === 'true',
};
```

### 2. Logging and Monitoring
```javascript
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'esahayak-api.log' })
  ]
});

api.createStampOrder = new Proxy(api.createStampOrder, {
  apply: async function(target, thisArg, argumentsList) {
    const startTime = Date.now();
    const [orderData] = argumentsList;

    logger.info('Creating stamp order', {
      state: orderData.state.name,
      amount: orderData.stampAmount
    });

    try {
      const result = await target.apply(thisArg, argumentsList);
      logger.info('Order created successfully', {
        orderId: result.orderId,
        duration: Date.now() - startTime
      });
      return result;
    } catch (error) {
      logger.error('Order creation failed', {
        error: error.message,
        duration: Date.now() - startTime
      });
      throw error;
    }
  }
});
```

### 3. Testing Framework
```javascript
describe('eSahayak API Integration', () => {
  let api;

  beforeEach(() => {
    api = new eSahayakAPI(process.env.TEST_API_KEY);
  });

  test('should get states successfully', async () => {
    const states = await api.getStates();
    expect(states.success).toBe(true);
    expect(states.data).toBeInstanceOf(Array);
    expect(states.data.length).toBeGreaterThan(0);
  });

  test('should create stamp order with valid data', async () => {
    const orderData = {
      // Valid test order data
    };

    const result = await api.createStampOrder(orderData);
    expect(result.success).toBe(true);
    expect(result.orderId).toBeDefined();
  });

  test('should handle insufficient credits gracefully', async () => {
    // Test with insufficient credits
    const orderData = { stampAmount: 999999 }; // Very high amount

    await expect(api.createStampOrder(orderData))
      .rejects.toThrow('Insufficient credits');
  });
});
```

For more examples and advanced patterns, visit our [GitHub repository](https://github.com/esahayak/api-examples).
