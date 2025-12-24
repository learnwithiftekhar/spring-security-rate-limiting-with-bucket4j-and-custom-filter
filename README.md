# Spring Security Rate Limiting with Bucket4j and Redis

This project demonstrates how to implement distributed rate limiting in a Spring Boot application using Spring Security, Bucket4j, and Redis. It provides a robust way to protect your API from abuse by limiting the number of requests a client can make within a specific timeframe.

## Features

- **Distributed Rate Limiting**: Uses Redis as a centralized storage for rate limit buckets, making it suitable for multi-instance (clustered) environments.
- **Client Identification**: Extracts client IP addresses, with support for `X-Forwarded-For` headers to correctly identify clients behind proxies or load balancers.
- **Spring Security Integration**: Implemented as a custom `OncePerRequestFilter` within the Spring Security filter chain.
- **Customizable Quotas**: Easily configurable request limits (e.g., 10 requests per minute).
- **Graceful Error Handling**: Returns a structured JSON response with `429 Too Many Requests` status and a `X-Rate-Limit-Retry-After-Seconds` header when the quota is exceeded.

## Prerequisites

- **Java 17** or higher
- **Maven 3.6+**
- **Redis Server** (Running on `localhost:6379` by default)

## Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/learnwithiftekhar/spring-security-rate-limiting-with-bucket4j-and-custom-filter.git
cd spring-security-rate-limiting-with-bucket4j-and-custom-filter
```

### 2. Configure Redis
Update `src/main/resources/application.yml` if your Redis server is running on a different host or port:
```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

### 3. Build and Run
```bash
mvn clean install
mvn spring-boot:run
```

## How It Works

### 1. Redis Configuration (`RateLimitConfig.java`)
We use the Lettuce driver to connect to Redis. A `ProxyManager` is configured to manage the state of Bucket4j buckets in Redis. We also define a TTL (Time-To-Live) for unused buckets to prevent Redis from filling up with old IP data.

### 2. Rate Limiting Service (`RateLimitingService.java`)
This service is responsible for resolving the bucket for a given key (client IP). If a bucket doesn't exist for an IP, it creates one with the defined configuration (e.g., 10 tokens per minute).

### 3. Rate Limit Filter (`RateLimitFilter.java`)
This filter intercepts every request:
1. Extracts the client's IP address.
2. Retrieves or creates the corresponding bucket from Redis.
3. Attempts to consume one token.
4. If successful, it adds the `X-Rate-Limit-Remaining` header.
5. If unsuccessful (quota exceeded), it returns a `429` status code with a JSON body and the `X-Rate-Limit-Retry-After-Seconds` header.

## API Documentation

The rate limit is currently set to **10 requests per minute** per IP address.

### Example Success Response
**Headers:**
```text
HTTP/1.1 200 OK
X-Rate-Limit-Remaining: 9
```

### Example Rate Limited Response
**Status:** `429 Too Many Requests`  
**Headers:**
```text
X-Rate-Limit-Retry-After-Seconds: 45
Content-Type: application/json
```
**Body:**
```json
{
    "status": 429,
    "error": "Too Many Requests",
    "message": "You have exhausted your API Request Quota",
    "retryAfterSeconds": 45
}
```

## Connect with Me

- **LinkedIn**: [Hossain Md Iftekhar](https://www.linkedin.com/in/hossain-md-iftekhar/)
- **YouTube**: [Learn with Ifte](https://www.youtube.com/@learnWithIfte?sub_confirmation=1)
- **Email**: learnwithiftekhar@gmail.com

## License
This project is open-source and available under the [MIT License](LICENSE).
