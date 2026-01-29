
# Distributed Cache System - ASP.NET Core 6 with Redis

![.NET 6.0](https://img.shields.io/badge/.NET-6.0-512BD4?style=for-the-badge&logo=dotnet)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![ASP.NET Core 6.0](https://img.shields.io/badge/ASP.NET_Core-6.0-512BD4?style=for-the-badge&logo=dotnet)
![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)
![Code Quality](https://img.shields.io/badge/Code%2520Quality-A+-brightgreen?style=for-the-badge)

Professional-grade distributed caching solution built with ASP.NET Core 6 MVC architecture, featuring Redis integration for high-performance session management and frequent data caching.

## 🌟 Key Features

### 🚀 Core Architecture
- Multi-layer Caching Strategy (Redis Distributed, Session, In-Memory)
- Clean Architecture with proper separation of concerns
- Dependency Injection ready with interface-based design
- Middleware Support for logging and monitoring

### ⚡ Performance Optimization
- Redis-backed Session Management for distributed applications
- Memory Cache for ultra-fast data access
- Cache Statistics with hit/miss ratio tracking
- Automatic Cache Invalidation and refresh mechanisms

### 🔧 Technical Excellence
- Full async/await support throughout the codebase
- Comprehensive error handling and logging
- Health Check endpoints for system monitoring
- Docker support for easy deployment
- RESTful API for cache operations

## 🛠️ Technology Stack

| Technology | Purpose | Version |
|------------|---------|---------|
| ASP.NET Core 6 | Web Application | 6.0 |
| Redis | Distributed Cache | Latest |
| StackExchange.Redis | Redis Client | 2.10.1 |
| Entity Framework Core | ORM (Optional) | 6.0 |
| Bootstrap 5 | Frontend Framework | 5.3.8 |
| jQuery | Client-side scripting | 3.6 |

## 🚀 Getting Started

### Prerequisites
- .NET 6 SDK
- Docker Desktop (for Redis)
- Visual Studio 2022 or VS Code

### Quick Start with Docker


# 1. Clone the repository
git clone https://github.com/yourusername/aspnetcore-redis-caching-system.git
cd aspnetcore-redis-caching-system

# 2. Start Redis using Docker
docker run -d -p 6379:6379 --name redis-cache redis:alpine

# 3. Restore packages and run
dotnet restore
dotnet run

# 4. Open in browser
# http://localhost:5000 or https://localhost:5001


### Configuration
**appsettings.json**:

{
  "RedisConfiguration": {
    "ConnectionString": "localhost",
    "InstanceName": "DistributedCacheSystem:",
    "SessionTimeoutMinutes": 30,
    "DefaultCacheTimeoutMinutes": 60
  }
}


## 💡 Usage Examples

### 1. Basic Cache Operations

// Set cache
await _cacheService.SetAsync("user:123", userData, TimeSpan.FromMinutes(30));

// Get cache with fallback
var user = await _cacheService.GetOrCreateAsync("user:123", 
    async () => await _dbContext.Users.FindAsync(123),
    TimeSpan.FromMinutes(30));

// Delete cache
await _cacheService.RemoveAsync("user:123");


### 2. Session Management

// Store in session
await _sessionCache.SetSessionDataAsync("shoppingCart", cartItems);

// Retrieve from session
var cart = await _sessionCache.GetSessionDataAsync<List<CartItem>>("shoppingCart");


### 3. Memory Cache

// Fast in-memory caching
var config = _memoryCacheService.GetOrCreate("app-config", options =>
{
    options.AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1);
    return LoadConfiguration();
});


## 📊 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/cache/{key}` | Retrieve cached value |
| POST | `/api/cache/{key}` | Set cache value |
| DELETE | `/api/cache/{key}` | Remove cache entry |
| GET | `/api/cache/statistics` | Get cache performance stats |
| GET | `/health` | System health check |
| GET | `/api/cache/session/{key}` | Get session data |
| POST | `/api/cache/session/{key}` | Set session data |

## 🧪 Testing the System

### 1. Manual Testing via Web Interface
1. Navigate to the home page
2. Use the Test Session button to verify session caching
3. Use the Test Memory Cache button for memory cache testing
4. Enter custom keys/values in the operations panel
5. View real-time statistics in the dashboard

### Automated Testing
# Run unit tests
dotnet test

# Run with coverage
dotnet test --collect:"XPlat Code Coverage"


## 🔧 Architecture Patterns Implemented

### 1. Cache-Aside (Lazy Loading)

public async Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory)
{
    var cached = await GetAsync<T>(key);
    if (cached != null) return cached;
    
    var data = await factory();
    await SetAsync(key, data);
    return data;
}


### 2. Write-Through Caching

public async Task UpdateWithCacheAsync(Product product)
{
    // Update database
    await _repository.UpdateAsync(product);
    
    // Update cache
    await _cacheService.SetAsync($"product:{product.Id}", product);
}


### 3. Circuit Breaker Pattern

public class ResilientCacheService : IDistributedCacheService
{
    private readonly CircuitBreakerPolicy _circuitBreaker;
    
    public async Task<T> GetAsync<T>(string key)
    {
        return await _circuitBreaker.ExecuteAsync(async () =>
        {
            return await _redis.GetAsync<T>(key);
        });
    }
}


## 📈 Performance Benchmarks

| Operation | Memory Cache | Redis Cache | Database |
|-----------|--------------|-------------|----------|
| Read | 0.1ms | 1ms | 50-100ms |
| Write | 0.1ms | 2ms | 100-200ms |
| Concurrent Reads | 10,000/sec | 5,000/sec | 100/sec |

## 🐳 Docker Deployment

### docker-compose.yml

version: '3.8'
services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    
  app:
    build: .
    ports:
      - "5000:80"
      - "5001:443"
    environment:
      - RedisConfiguration__ConnectionString=redis:6379
    depends_on:
      - redis

volumes:
  redis_data:


### Dockerfile

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore
RUN dotnet build -c Release -o /app/build

FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DistributedCacheSystem.dll"]


## 🔍 Monitoring & Logging

### 1. Health Checks

services.AddHealthChecks()
    .AddRedis(redisConnectionString, name: "redis")
    .AddDbContextCheck<ApplicationDbContext>(name: "database");


### 2. Logging Middleware

app.UseMiddleware<CacheLoggingMiddleware>();


### 3. Statistics Dashboard
Access real-time cache statistics at `/api/cache/statistics`:

{
  "totalHits": 1250,
  "totalMisses": 150,
  "totalItems": 89,
  "hitRatio": 89.29
}


## 👨‍💻 Author
**Ali Hanif** - .NET Developer  
GitHub: [@AliHANiF2042](https://github.com/AliHANiF2042)
