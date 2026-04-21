# Rate Limiter Middleware

A simple Express.js middleware that implements rate limiting to restrict the number of API requests per user within a specified time window.

## Overview

This rate limiter middleware tracks incoming requests by user ID and enforces a limit of **5 requests per second**. Any requests exceeding this limit receive a 404 response with a "Rate limit" message.

## Installation

### Prerequisites
- Node.js installed on your system
- npm (Node Package Manager)

### Setup

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start the server:**
   ```bash
   npm start
   ```
   Or run directly:
   ```bash
   node ratelimiter.js
   ```

The server will start on `http://localhost:3000`

## How It Works

### Rate Limiting Logic

The rate limiter operates by:

1. **Tracking Requests**: Maintains a counter (`numberOfRequestForUser`) for each user ID
2. **Time Window Reset**: Every 1 second, the counter resets to `{}` (empty object), clearing all request counts
3. **Request Validation**: For each incoming request:
   - Extracts the `user-id` from request headers
   - Increments the request count for that user
   - If count exceeds 5 in the current second window, returns HTTP 404 "Rate limit"
   - Otherwise, allows the request to proceed (`next()`)

### Key Features

- **Per-User Tracking**: Different users have independent request counters
- **Sliding Window Reset**: 1-second time window with automatic reset via `setInterval()`
- **Header-Based Identification**: Uses `user-id` header to identify users
- **Middleware Integration**: Applied globally to all routes using `app.use()`

## Code Breakdown

### 1. Initialize Express App
```javascript
const express = require('express');
const app = express();
```
Creates an Express application instance.

### 2. Request Counter Object
```javascript
let numberOfRequestForUser = {};
```
Object that stores request counts per user ID (e.g., `{ "user1": 3, "user2": 1 }`)

### 3. Reset Counter Every Second
```javascript
setInterval(function() {
    numberOfRequestForUser = {};
}, 1000);
```
Resets all counters to zero every 1000ms (1 second), creating the time window for rate limiting.

### 4. Rate Limiter Middleware
```javascript
app.use(function(req, res, next) {
    const userId = req.headers['user-id'];
    
    if(numberOfRequestForUser[userId]) {
        numberOfRequestForUser[userId]++;
        
        if(numberOfRequestForUser[userId] > 5) {
            res.status(404).send("Rate limit");
        } else {
            next();
        }
    } else {
        numberOfRequestForUser[userId] = 1;
        next();
    }
});
```

**Flow:**
- Get user ID from request headers
- If user exists in counter:
  - Increment count
  - If count > 5: Send 404 error (rate limit exceeded)
  - Else: Allow request to continue
- If user is new:
  - Initialize count to 1
  - Allow request to continue

### 5. Sample Route
```javascript
app.get('/user', function(req, res) {
    res.status(200).json({name: "Mathew"});
});
```
A simple test endpoint that returns a JSON response.

### 6. Start Server
```javascript
app.listen(3000);
```
Server listens on port 3000.

## Usage

### Making Requests

Send requests with a `user-id` header:

```bash
# Using curl
curl -H "user-id: user123" http://localhost:3000/user

# Requests 1-5: Success (200 OK)
# Response: {"name":"Mathew"}

# Request 6+: Rate limit exceeded (404)
# Response: "Rate limit"
```

### Example with Different Users

```bash
# User 1: First request
curl -H "user-id: alice" http://localhost:3000/user
# ✓ Success

# User 2: First request (different user, separate counter)
curl -H "user-id: bob" http://localhost:3000/user
# ✓ Success

# User 1: Requests 2-5
# ✓ Success for each

# User 1: Request 6 (within same second)
curl -H "user-id: alice" http://localhost:3000/user
# ✗ Rate limit error (404)
```

## Important Notes

⚠️ **Production Limitations:**
- This implementation stores data in **memory only** (lost on server restart)
- For production, use Redis or MongoDB for persistent rate limit storage
- The implementation doesn't handle:
  - Different rate limits for different endpoints
  - User authentication beyond header-based ID
  - Custom error responses
  - Graceful degradation under high load

## Configuration

To modify rate limits, edit these values in `ratelimiter.js`:

```javascript
// Change the time window (currently 1000ms = 1 second)
}, 1000);  // Modify this value

// Change the request limit (currently 5 requests)
if(numberOfRequestForUser[userId] > 5) {  // Modify 5 to desired limit
```

## Project Structure

```
RatelimiterMiddleware/
├── package.json          # Project dependencies and scripts
├── ratelimiter.js        # Main middleware implementation
└── README.md             # This file
```

## Dependencies

- **express**: ^5.2.1 - Web application framework

## License

ISC

---

**Created**: 2026
**Type**: CommonJS Module
