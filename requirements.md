# 1. User Authentication System

      API Endpoints
      | Method | Endpoint               | Description                     |
      |--------|------------------------|---------------------------------|
      | POST   | `/api/v1/auth/register`| Guest/Host registration         |
      | POST   | `/api/v1/auth/login`   | JWT token generation            |
      | POST   | `/api/v1/auth/forgot-password` | Password reset initiation |
      
      Input/Output Specifications
      Registration (POST /register)
      ```json
      // Request
      {
        "email": "user@example.com",
        "password": "SecurePass123!",
        "role": "guest|host",
        "phone": "+1234567890" // Optional
      }
      
      // Response (201 Created)
      {
        "id": "usr_abc123",
        "email": "user@example.com",
        "role": "guest",
        "verification_token": "tok_xyz456"
      }
      
      Validation Rules
      - Email: Valid format + DNS MX record check
      - Password: 12+ chars, 1+ uppercase, 1+ number, 1+ special char
      - Role: Enum(guest, host, admin)
      - Rate Limiting: 5 requests/minute/IP
      
       Performance Criteria
      - Response time: <500ms p95
      - Concurrent users: 10,000
      - Uptime: 99.99% SLA
      
      # 2. Property Management
      
      API Endpoints
      | Method | Endpoint               | Description                     |
      |--------|------------------------|---------------------------------|
      | POST   | `/api/v1/properties`   | Create new property             |
      | PUT    | `/api/v1/properties/{id}` | Update property              |
      | GET    | `/api/v1/properties/search` | Filtered property search   |
      
      Input/Output Specifications
       Property Creation (POST /properties)
      ```json
      // Request
      {
        "host_id": "usr_abc123",
        "title": "Beachfront Villa",
        "description": "Luxury 3BR...",
        "price_per_night": 350.00,
        "amenities": ["wifi", "pool"],
        "location": {
          "lat": 40.7128,
          "lng": -74.0060
        }
      }
      
      // Response (201 Created)
      {
        "id": "prop_xyz789",
        "calendar_updated_at": "2023-08-20T12:00:00Z"
      }
      ```
      
      Validation Rules
      - Price: >0, decimal(10,2)
      - Location: Valid geo-coordinates (-90≤lat≤90, -180≤lng≤180)
      - Media: 5-20 images (1-5MB each, JPEG/PNG)
      - Ownership: Host must own the property to edit
      
      Performance Criteria
      - Search response: <1s for 10,000 properties
      - Image processing: <2s per image
      - Availability checks: <100ms
      
      ---
      
      # 3. Booking System
      
      API Endpoints
      | Method | Endpoint               | Description                     |
      |--------|------------------------|---------------------------------|
      | POST   | `/api/v1/bookings`     | Create reservation              |
      | GET    | `/api/v1/bookings/{id}`| Retrieve booking details        |
      | DELETE | `/api/v1/bookings/{id}`| Cancel booking                  |
      
      Input/Output Specifications
      Booking Creation (POST /bookings)
      ```json
      // Request
      {
        "property_id": "prop_xyz789",
        "user_id": "usr_abc123",
        "start_date": "2023-10-01",
        "end_date": "2023-10-07",
        "payment_method": "stripe"
      }
      
      // Response (201 Created)
      {
        "id": "book_123abc",
        "total": 2450.00,
        "currency": "USD",
        "payment_due_by": "2023-09-28T23:59:59Z"
      }
      ```
      
      Validation Rules
      - Date Conflict: Atomic check against bookings table
      - Pricing: 
        ```sql
        SELECT price_per_night * (end_date - start_date) 
        FROM properties WHERE id = ?
        ```
      - Cancellation: Full refund if >14 days before check-in
      
      Performance Criteria
      - Booking creation: <800ms p99
      - Payment processing: <5s timeout
      - Calendar updates: Real-time (<100ms propagation)
      
      ---
      
      # Technical Standards
      1. Error Codes:
         - 429 for rate limits
         - 423 for locked resources
         - 451 for legal restrictions
      
      2. Idempotency:
         - All POST endpoints accept `Idempotency-Key` header
      
      3. Monitoring:
         - Datadog dashboards for:
           - Auth success/failure rates
           - Booking conversion funnel
           - Payment processing latency
