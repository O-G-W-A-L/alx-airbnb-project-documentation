# Requirements

## 1. User Authentication

### Endpoints
- **POST** `/api/v1/auth/register`
- **POST** `/api/v1/auth/login`
- **POST** `/api/v1/auth/logout`
- **GET**  `/api/v1/auth/refresh`

### Request / Response
- **Register**  
  - Request:
    ```json
    {
      "first_name": "string",
      "last_name":  "string",
      "email":      "email",
      "password":   "string"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "user_id":   "UUID",
      "email":     "email",
      "created_at":"timestamp"
    }
    ```
- **Login**  
  - Request:
    ```json
    {
      "email":    "email",
      "password": "string"
    }
    ```
  - Response: `200 OK`
    ```json
    {
      "access_token":"JWT",
      "expires_in":   3600
    }
    ```

### Validation
- Email must be unique and valid format  
- Password ≥ 12 chars, mixed case, digits, symbols  
- Rate limit login: 5 attempts/min per IP

### Performance & Security
- TLS required  
- JWT signed with RS256 (2048-bit key)  
- Response time < 200 ms (95th percentile at 100 RPS)

---

## 2. Property Management

### Endpoints
- **POST**   `/api/v1/properties`
- **GET**    `/api/v1/properties`
- **GET**    `/api/v1/properties/{id}`
- **PUT**    `/api/v1/properties/{id}`
- **DELETE** `/api/v1/properties/{id}`

### Request / Response
- **Create**  
  - Request:
    ```json
    {
      "host_id":         "UUID",
      "name":            "string",
      "description":     "string",
      "location":        "string",
      "price_per_night": 0.00,
      "amenities":       ["string"]
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "property_id":"UUID",
      "created_at": "timestamp"
    }
    ```
- **List/Search**  
  - Query params: `location`, `min_price`, `max_price`, `amenities`, `page`, `limit`  
  - Response: `200 OK`
    ```json
    {
      "data": [
        { "property_id":"...", "name":"...", "price_per_night":0 }
      ],
      "meta": { "page":1, "limit":20, "total":123 }
    }
    ```

### Validation
- Name: 5–100 chars  
- Location: non-empty, ≤ 200 chars  
- Price_per_night: positive, two decimals max

### Performance
- Default `limit=20`, max `limit=100`  
- Index on `location`, `price_per_night`  
- Rate limit: 50 req/min per user

---

## 3. Booking System

### Endpoints
- **POST**   `/api/v1/bookings`
- **GET**    `/api/v1/bookings`
- **GET**    `/api/v1/bookings/{id}`
- **PATCH**  `/api/v1/bookings/{id}/cancel`
- **PUT**    `/api/v1/bookings/{id}`

### Request / Response
- **Create**  
  - Request:
    ```json
    {
      "property_id":"UUID",
      "user_id":    "UUID",
      "start_date":"YYYY-MM-DD",
      "end_date":  "YYYY-MM-DD"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "booking_id":"UUID",
      "total_price":0.00,
      "status":"pending"
    }
    ```

### Validation
- `start_date` < `end_date`  
- No overlapping bookings on the same property  
- Status ∈ {pending, confirmed, canceled, completed}

### Performance & Integrity
- Atomic transaction to prevent double bookings  
- Response time < 250 ms (95th percentile at 200 RPS)  
- Trigger notifications on status change

---

## 4. Payment Integration

### Endpoints
- **POST** `/api/v1/payments`

### Request / Response
- **Create**  
  - Request:
    ```json
    {
      "booking_id":     "UUID",
      "amount":         0.00,
      "payment_method":"credit_card|paypal|stripe"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "payment_id":"UUID",
      "status":"processed",
      "payment_date":"timestamp"
    }
    ```

### Validation
- Amount matches booking total  
- Method ∈ {credit_card, paypal, stripe}

### Performance & Security
- PCI-compliant gateway integration  
- Response time < 300 ms (95th percentile at 100 RPS)  
- Record payment atomic with booking confirmation

