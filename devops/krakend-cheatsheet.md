# KrakenD Cheatsheet

KrakenD is a high-performance, stateless API Gateway that enables you to
create a unified API layer by aggregating, filtering, and manipulating multiple
backend services. It's designed for microservices architectures with
ultra-high performance and zero dependencies.

## Overview

### Key Features

- **High Performance** - Ultra-fast API Gateway with sub-millisecond latency
- **Stateless** - No database required, configuration-driven
- **Backend Aggregation** - Combine multiple backend services into single endpoints
- **Data Manipulation** - Transform, filter, and aggregate response data
- **Security** - Authentication, authorization, rate limiting, CORS
- **Caching** - Multiple caching layers for optimal performance
- **Circuit Breaker** - Fault tolerance and resilience patterns
- **Load Balancing** - Multiple load balancing strategies
- **Monitoring** - Metrics, logging, and observability
- **Plugin System** - Extensible with custom middleware
- **Protocol Support** - HTTP/HTTPS, HTTP/2, WebSockets, gRPC
- **Zero Dependencies** - Single binary deployment

### Architecture

```text
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps  │    │   KrakenD API   │    │   Backend       │
│                 │    │     Gateway     │    │   Services      │
│ • Web Apps      │◄──►│                 │◄──►│                 │
│ • Mobile Apps   │    │ • Aggregation   │    │ • Microservice1 │
│ • IoT Devices   │    │ • Transformation│    │ • Microservice2 │
│ • Third Party   │    │ • Security      │    │ • Database APIs │
└─────────────────┘    │ • Rate Limiting │    │ • Legacy APIs   │
                       │ • Caching       │    │ • External APIs │
                       └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   Middleware    │
                       │                 │
                       │ • Auth Plugins  │
                       │ • Metrics       │
                       │ • Logging       │
                       │ • Custom Logic  │
                       └─────────────────┘
```

## Installation

### macOS Installation

```bash
# Using Homebrew
brew tap krakendio/tap
brew install krakend

# Using Binary Download
curl -L \
  https://github.com/krakendio/krakend-ce/releases/latest/download/krakend_darwin_amd64.tar.gz | \
  tar xz
sudo mv krakend /usr/local/bin/

# For Apple Silicon (M1/M2)
curl -L \
  https://github.com/krakendio/krakend-ce/releases/latest/download/krakend_darwin_arm64.tar.gz | \
  tar xz
sudo mv krakend /usr/local/bin/

# Verify installation
krakend version
```

### Linux Installation

```bash
# Ubuntu/Debian
wget https://repo.krakend.io/bin/krakend_2.4.1_amd64_generic-linux.tar.gz
tar -xzf krakend_2.4.1_amd64_generic-linux.tar.gz
sudo mv krakend /usr/local/bin/

# Using package manager (Ubuntu/Debian)
echo "deb https://repo.krakend.io/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/krakend.list
wget -O - https://repo.krakend.io/signing.key | sudo apt-key add -
sudo apt update
sudo apt install krakend

# CentOS/RHEL
echo '[krakend]
name=KrakenD
baseurl=https://repo.krakend.io/rpm
gpgcheck=1
enabled=1
gpgkey=https://repo.krakend.io/signing.key' | sudo tee /etc/yum.repos.d/krakend.repo
sudo yum install krakend

# Using systemd service
sudo cat > /etc/systemd/system/krakend.service << EOF
[Unit]
Description=KrakenD API Gateway
After=network.target

[Service]
Type=simple
User=krakend
Group=krakend
ExecStart=/usr/local/bin/krakend run -c /etc/krakend/krakend.json
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=krakend
KillMode=mixed
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target
EOF

# Create user and directories
sudo useradd -r -s /bin/false krakend
sudo mkdir -p /etc/krakend
sudo chown krakend:krakend /etc/krakend

# Enable and start service
sudo systemctl enable krakend
sudo systemctl start krakend
```

### Docker Installation

```bash
# Run KrakenD with Docker
docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend:2.4 run --config /etc/krakend/krakend.json

# Using Docker Compose
cat > docker-compose.yml << EOF
version: '3.8'

services:
  krakend:
    image: devopsfaith/krakend:2.4
    ports:
      - "8080:8080"
    volumes:
      - ./krakend.json:/etc/krakend/krakend.json:ro
      - ./templates:/etc/krakend/templates:ro
    command: ["run", "--config", "/etc/krakend/krakend.json"]
    restart: unless-stopped
    environment:
      - KRAKEND_PORT=8080
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/__health"]
      interval: 30s
      timeout: 10s
      retries: 3
EOF

docker-compose up -d

# Development with live reload
docker run -it --rm -p 8080:8080 \
  -v $PWD:/etc/krakend/ \
  devopsfaith/krakend:2.4 \
  run --config /etc/krakend/krakend.json --debug
```

### Kubernetes Installation

```yaml
# Create ConfigMap for configuration
kubectl create configmap krakend-config --from-file=krakend.json

# Deployment manifest
cat > krakend-deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krakend
  labels:
    app: krakend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: krakend
  template:
    metadata:
      labels:
        app: krakend
    spec:
      containers:
      - name: krakend
        image: devopsfaith/krakend:2.4
        ports:
        - containerPort: 8080
        command: ["krakend"]
        args: ["run", "--config", "/etc/krakend/krakend.json"]
        volumeMounts:
        - name: krakend-config
          mountPath: /etc/krakend
        env:
        - name: KRAKEND_PORT
          value: "8080"
        livenessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      volumes:
      - name: krakend-config
        configMap:
          name: krakend-config
---
apiVersion: v1
kind: Service
metadata:
  name: krakend-service
spec:
  selector:
    app: krakend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
EOF

kubectl apply -f krakend-deployment.yaml
```

## Basic Commands

### CLI Commands

```bash
# Start KrakenD server
krakend run --config krakend.json
krakend run -c krakend.json

# Start with debug mode
krakend run --config krakend.json --debug

# Start on specific port
krakend run --config krakend.json --port 9000

# Check configuration syntax
krakend check --config krakend.json
krakend check -c krakend.json

# Validate configuration
krakend validate --config krakend.json

# Generate configuration
krakend generate --help

# Audit configuration
krakend audit --config krakend.json

# Show version
krakend version

# Show help
krakend help
krakend run --help

# Environment variables
export KRAKEND_PORT=8080
export KRAKEND_DEBUG=true
export KRAKEND_CONFIG_PATH=/etc/krakend/
```

### Configuration Generation

```bash
# Generate basic configuration
krakend generate --file krakend.json

# Generate with specific options
krakend generate \
  --file krakend.json \
  --listen-port 8080 \
  --service-name "My API Gateway"

# Generate OpenAPI spec
krakend generate openapi --config krakend.json --out api-spec.json

# Generate Postman collection
krakend generate postman --config krakend.json --out collection.json
```

## Configuration Structure

### Basic Configuration

```json
{
  "version": 3,
  "name": "My API Gateway",
  "timeout": "3000ms",
  "cache_ttl": "300s",
  "port": 8080,
  "host": ["http://localhost:8080"],
  "endpoints": [
    {
      "endpoint": "/users/{id}",
      "method": "GET",
      "output_encoding": "json",
      "backend": [
        {
          "url_pattern": "/api/users/{id}",
          "encoding": "json",
          "sd": "static",
          "method": "GET",
          "host": ["http://backend-service:8000"]
        }
      ]
    }
  ]
}
```

### Advanced Configuration

```json
{
  "version": 3,
  "name": "Advanced API Gateway",
  "timeout": "3000ms",
  "cache_ttl": "300s",
  "port": 8080,
  "host": ["https://api.example.com"],
  "tls": {
    "public_key": "/path/to/cert.pem",
    "private_key": "/path/to/key.pem"
  },
  "extra_config": {
    "security/cors": {
      "allow_origins": ["*"],
      "allow_methods": ["GET", "POST", "PUT", "DELETE"],
      "allow_headers": ["Origin", "Authorization", "Content-Type"],
      "expose_headers": ["Content-Length"],
      "max_age": "12h",
      "allow_credentials": false
    },
    "telemetry/metrics": {
      "collection_time": "60s",
      "proxy_disabled": false,
      "router_disabled": false,
      "backend_disabled": false,
      "endpoint_disabled": false,
      "listen_address": ":8090"
    },
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true
    }
  },
  "endpoints": []
}
```

## Endpoint Configuration

### Basic Endpoint

```json
{
  "endpoint": "/v1/users/{id}",
  "method": "GET",
  "timeout": "5000ms",
  "cache_ttl": "300s",
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "encoding": "json",
      "method": "GET",
      "host": ["http://user-service:8001"]
    }
  ]
}
```

### Endpoint with Multiple Backends

```json
{
  "endpoint": "/v1/user-profile/{id}",
  "method": "GET",
  "timeout": "5000ms",
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "encoding": "json",
      "method": "GET",
      "host": ["http://user-service:8001"],
      "group": "user"
    },
    {
      "url_pattern": "/profiles/{id}",
      "encoding": "json",
      "method": "GET",
      "host": ["http://profile-service:8002"],
      "group": "profile"
    },
    {
      "url_pattern": "/orders/user/{id}/count",
      "encoding": "json",
      "method": "GET",
      "host": ["http://order-service:8003"],
      "group": "orders"
    }
  ]
}
```

### Endpoint with Data Manipulation

```json
{
  "endpoint": "/v1/users/{id}/summary",
  "method": "GET",
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "encoding": "json",
      "method": "GET",
      "host": ["http://user-service:8001"],
      "allow": ["id", "name", "email", "created_at"],
      "deny": ["password", "secret_key"],
      "mapping": {
        "user_id": "id",
        "user_name": "name",
        "user_email": "email"
      },
      "group": "user"
    }
  ],
  "extra_config": {
    "modifier/jmespath": {
      "expr": "{
        user_info: user,
        timestamp: `now()`
      }"
    }
  }
}
```

### POST/PUT Endpoints

```json
{
  "endpoint": "/v1/users",
  "method": "POST",
  "input_headers": ["Content-Type", "Authorization"],
  "input_query_strings": ["validate"],
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/users",
      "encoding": "json",
      "method": "POST",
      "host": ["http://user-service:8001"]
    }
  ]
}
```

## Backend Configuration

### Basic Backend

```json
{
  "url_pattern": "/api/users/{id}",
  "encoding": "json",
  "method": "GET",
  "host": ["http://backend1:8000", "http://backend2:8000"],
  "sd": "static"
}
```

### Backend with Load Balancing

```json
{
  "url_pattern": "/api/users/{id}",
  "encoding": "json",
  "method": "GET",
  "host": [
    "http://backend1:8000",
    "http://backend2:8000",
    "http://backend3:8000"
  ],
  "extra_config": {
    "backend/http": {
      "return_error_details": "backend_alias"
    },
    "qos/ratelimit/proxy": {
      "max_rate": 100,
      "capacity": 100
    },
    "qos/circuit-breaker": {
      "interval": 60,
      "timeout": 10,
      "max_errors": 5,
      "name": "cb-backend-1",
      "log_status_change": true
    }
  }
}
```

### Backend with Authentication

```json
{
  "url_pattern": "/api/users/{id}",
  "encoding": "json",
  "method": "GET",
  "host": ["http://secure-backend:8000"],
  "extra_config": {
    "auth/api-keys": {
      "key": "X-API-Key",
      "location": "header"
    },
    "modifier/martian": {
      "header.Modifier": {
        "scope": ["request"],
        "name": "Authorization",
        "value": "Bearer {{.JWT_TOKEN}}"
      }
    }
  }
}
```

## Data Transformation

### Response Filtering

```json
{
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "host": ["http://user-service:8001"],
      "allow": ["id", "name", "email", "profile"],
      "deny": ["password", "secret", "internal_notes"]
    }
  ]
}
```

### Field Mapping

```json
{
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "host": ["http://user-service:8001"],
      "mapping": {
        "user_id": "id",
        "full_name": "name",
        "email_address": "email",
        "creation_date": "created_at"
      }
    }
  ]
}
```

### Response Grouping

```json
{
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "host": ["http://user-service:8001"],
      "group": "user_data"
    },
    {
      "url_pattern": "/profiles/{id}",
      "host": ["http://profile-service:8002"],
      "group": "profile_data"
    }
  ]
}
```

### JMESPath Expressions

```json
{
  "extra_config": {
    "modifier/jmespath": {
      "expr": "{
        user: user_data,
        profile: profile_data,
        summary: {
          id: user_data.id,
          name: user_data.name,
          avatar: profile_data.avatar_url,
          last_login: profile_data.last_login
        },
        timestamp: `now()`
      }"
    }
  }
}
```

### Response Templates

```json
{
  "extra_config": {
    "modifier/response-body-generator": {
      "template": "{
        \"status\": \"success\",
        \"data\": {
          \"user\": {{marshal .user_data}},
          \"profile\": {{marshal .profile_data}}
        },
        \"meta\": {
          \"timestamp\": \"{{now}}\",
          \"version\": \"v1\"
        }
      }",
      "content_type": "application/json",
      "debug": false
    }
  }
}
```

## Security

### JWT Authentication

```json
{
  "extra_config": {
    "auth/validator": {
      "alg": "RS256",
      "jwk_url": "https://your-auth-server.com/.well-known/jwks.json",
      "audience": ["your-api"],
      "issuer": "https://your-auth-server.com",
      "cache": true,
      "cache_duration": 900,
      "disable_jwk_security": false,
      "jwk_fingerprints": [],
      "roles_key": "roles",
      "roles": ["user", "admin"]
    }
  }
}
```

### API Key Authentication

```json
{
  "extra_config": {
    "auth/api-keys": {
      "keys": [
        {
          "key": "api-key-1",
          "strategy": "header",
          "field": "X-API-Key",
          "roles": ["user"]
        },
        {
          "key": "admin-key-1", 
          "strategy": "query",
          "field": "api_key",
          "roles": ["admin"]
        }
      ]
    }
  }
}
```

### Basic Authentication

```json
{
  "extra_config": {
    "auth/basic": {
      "users": {
        "user1": "password1",
        "admin": "admin123"
      }
    }
  }
}
```

### CORS Configuration

```json
{
  "extra_config": {
    "security/cors": {
      "allow_origins": [
        "https://example.com",
        "https://app.example.com"
      ],
      "allow_methods": [
        "GET",
        "POST",
        "PUT",
        "DELETE",
        "OPTIONS"
      ],
      "allow_headers": [
        "Origin",
        "Authorization",
        "Content-Type",
        "X-Requested-With"
      ],
      "expose_headers": [
        "Content-Length",
        "X-Request-Id"
      ],
      "max_age": "12h",
      "allow_credentials": true,
      "debug": false
    }
  }
}
```

### Security Headers

```json
{
  "extra_config": {
    "security/http": {
      "allowed_hosts": ["api.example.com"],
      "ssl_redirect": true,
      "ssl_host": "api.example.com",
      "sts_seconds": 31536000,
      "sts_include_subdomains": true,
      "frame_deny": true,
      "content_type_nosniff": true,
      "browser_xss_filter": true,
      "content_security_policy": "default-src 'self'",
      "public_key_pins": [
        "pin-sha256=base64+primary==",
        "pin-sha256=base64+backup=="
      ],
      "public_key_pins_report_only": false,
      "hp_public_key_pins": false,
      "referrer_policy": "same-origin"
    }
  }
}
```

## Rate Limiting

### Global Rate Limiting

```json
{
  "extra_config": {
    "qos/ratelimit/service": {
      "max_rate": 1000,
      "capacity": 1000,
      "default_user_quota": 100,
      "strategy": "ip"
    }
  }
}
```

### Endpoint Rate Limiting

```json
{
  "endpoint": "/v1/users",
  "method": "GET",
  "extra_config": {
    "qos/ratelimit/router": {
      "max_rate": 100,
      "capacity": 100,
      "strategy": "header",
      "key": "X-User-ID"
    }
  }
}
```

### Backend Rate Limiting

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "host": ["http://user-service:8001"],
      "extra_config": {
        "qos/ratelimit/proxy": {
          "max_rate": 50,
          "capacity": 50
        }
      }
    }
  ]
}
```

### Distributed Rate Limiting (Redis)

```json
{
  "extra_config": {
    "qos/ratelimit/service": {
      "max_rate": 1000,
      "capacity": 1000,
      "strategy": "header",
      "key": "X-User-ID",
      "redis_url": "redis://localhost:6379",
      "tokenizer": "ip",
      "tokenizer_field": "X-Forwarded-For"
    }
  }
}
```

## Caching

### Memory Caching

```json
{
  "endpoint": "/v1/users/{id}",
  "cache_ttl": "300s",
  "backend": [
    {
      "url_pattern": "/users/{id}",
      "host": ["http://user-service:8001"]
    }
  ]
}
```

### Redis Caching

```json
{
  "extra_config": {
    "qos/http-cache": {
      "ttl": "300s",
      "cache_control": true,
      "etag": true,
      "redis": {
        "network": "tcp",
        "address": "localhost:6379",
        "password": "",
        "db": 0
      }
    }
  }
}
```

### CDN Caching

```json
{
  "extra_config": {
    "qos/http-cache": {
      "ttl": "3600s",
      "cache_control": true,
      "etag": true,
      "vary": ["Accept-Encoding", "User-Agent"],
      "cdn": {
        "provider": "cloudflare",
        "ttl": "86400s"
      }
    }
  }
}
```

## Circuit Breaker

### Basic Circuit Breaker

```json
{
  "backend": [
    {
      "url_pattern": "/api/users/{id}",
      "host": ["http://user-service:8001"],
      "extra_config": {
        "qos/circuit-breaker": {
          "interval": 60,
          "timeout": 10,
          "max_errors": 5,
          "name": "user-service-cb",
          "log_status_change": true
        }
      }
    }
  ]
}
```

### Advanced Circuit Breaker

```json
{
  "extra_config": {
    "qos/circuit-breaker": {
      "interval": 60,
      "timeout": 10,
      "max_errors": 5,
      "name": "advanced-cb",
      "log_status_change": true,
      "fallback_response": {
        "status": 503,
        "body": {
          "error": "Service temporarily unavailable",
          "retry_after": 60
        },
        "headers": {
          "Content-Type": "application/json",
          "Retry-After": "60"
        }
      }
    }
  }
}
```

## Monitoring and Observability

### Metrics Configuration

```json
{
  "extra_config": {
    "telemetry/metrics": {
      "collection_time": "60s",
      "proxy_disabled": false,
      "router_disabled": false,
      "backend_disabled": false,
      "endpoint_disabled": false,
      "listen_address": ":8090"
    }
  }
}
```

### Prometheus Integration

```json
{
  "extra_config": {
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 5,
      "enabled_layers": {
        "backend": true,
        "router": true,
        "pipe": true
      },
      "exporters": {
        "prometheus": {
          "port": 9090,
          "namespace": "krakend",
          "tag_host": false,
          "tag_path": true,
          "tag_method": true,
          "tag_statuscode": false
        }
      }
    }
  }
}
```

### Jaeger Tracing

```json
{
  "extra_config": {
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 5,
      "enabled_layers": {
        "backend": true,
        "router": true,
        "pipe": true
      },
      "exporters": {
        "jaeger": {
          "endpoint": "http://jaeger:14268/api/traces",
          "service_name": "krakend",
          "buffer_max_count": 1000
        }
      }
    }
  }
}
```

### Logging Configuration

```json
{
  "extra_config": {
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true,
      "format": "custom",
      "custom_format": "%{time:2006-01-02 15:04:05} %{level} %{message}\n"
    },
    "telemetry/gelf": {
      "address": "logstash:12201",
      "enable_tcp": false
    },
    "telemetry/logstash": {
      "address": "logstash:5044"
    }
  }
}
```

### Health Check

```json
{
  "extra_config": {
    "github.com/devopsfaith/krakend-health": {
      "path": "/__health",
      "cache_ttl": "300s",
      "checks": [
        {
          "name": "user-service",
          "url": "http://user-service:8001/health",
          "timeout": "3s",
          "interval": "30s",
          "method": "GET",
          "headers": {
            "User-Agent": "KrakenD Health Checker"
          }
        }
      ]
    }
  }
}
```

## Load Balancing

### Round Robin (Default)

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "host": [
        "http://user-service-1:8001",
        "http://user-service-2:8001",
        "http://user-service-3:8001"
      ]
    }
  ]
}
```

### Service Discovery with Consul

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "sd": "consul",
      "host": ["consul.service.consul:8500"],
      "extra_config": {
        "backend/consul": {
          "service": "user-service",
          "tag": "api",
          "health": true
        }
      }
    }
  ]
}
```

### Service Discovery with Eureka

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "sd": "eureka",
      "host": ["http://eureka:8761/eureka"],
      "extra_config": {
        "backend/eureka": {
          "service": "USER-SERVICE"
        }
      }
    }
  ]
}
```

### DNS Service Discovery

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "sd": "dns",
      "host": ["user-service.default.svc.cluster.local:8001"]
    }
  ]
}
```

## Advanced Features

### Request/Response Modification

```json
{
  "extra_config": {
    "modifier/martian": {
      "header.Modifier": {
        "scope": ["request", "response"],
        "name": "X-Custom-Header",
        "value": "Custom Value"
      },
      "query.Modifier": {
        "scope": ["request"],
        "name": "version",
        "value": "v2"
      },
      "body.Modifier": {
        "scope": ["request"],
        "contentType": "application/json",
        "body": "eyJhZGRpdGlvbmFsX2ZpZWxkIjoidmFsdWUifQ=="
      }
    }
  }
}
```

### GraphQL Support

```json
{
  "endpoint": "/graphql",
  "method": "POST",
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/graphql",
      "encoding": "json",
      "method": "POST",
      "host": ["http://graphql-server:4000"]
    }
  ],
  "extra_config": {
    "backend/graphql": {
      "type": "query",
      "query": "query GetUser($id: ID!) { user(id: $id) { id name email } }",
      "variables": {
        "id": "{id}"
      }
    }
  }
}
```

### gRPC Backend

```json
{
  "backend": [
    {
      "url_pattern": "/user.UserService/GetUser",
      "encoding": "grpc",
      "method": "POST",
      "host": ["grpc://user-service:9000"],
      "extra_config": {
        "backend/grpc": {
          "input_mapping": {
            "user_id": "id"
          },
          "output_mapping": {
            "user_data": "user"
          },
          "response_naming_convention": "snake_case",
          "use_request_id": true
        }
      }
    }
  ]
}
```

### WebSocket Proxy

```json
{
  "endpoint": "/ws",
  "method": "GET",
  "output_encoding": "no-op",
  "backend": [
    {
      "url_pattern": "/websocket",
      "encoding": "no-op",
      "method": "GET",
      "host": ["ws://websocket-service:8080"]
    }
  ],
  "extra_config": {
    "backend/websocket": {
      "input_headers": ["Sec-WebSocket-Protocol"],
      "connect_event": true,
      "disconnect_event": true,
      "message_event": true,
      "return_error_details": true
    }
  }
}
```

### Lambda Functions

```json
{
  "backend": [
    {
      "url_pattern": "/lambda",
      "encoding": "json",
      "method": "POST",
      "host": ["lambda://us-east-1"],
      "extra_config": {
        "backend/lambda": {
          "function_name": "my-function",
          "region": "us-east-1",
          "max_retries": 3,
          "endpoint": ""
        }
      }
    }
  ]
}
```

## Plugin Development

### Custom Middleware Plugin

```go
// main.go
package main

import (
    "context"
    "fmt"
    "net/http"
    "github.com/luraproject/lura/v2/config"
    "github.com/luraproject/lura/v2/proxy"
)

// HandlerRegisterer registers the custom middleware
type HandlerRegisterer interface {
    RegisterHandlers(f func(
        name string,
        handler func(
            context.Context,
            map[string]interface{},
        ) (http.Handler, error),
    ))
}

// RegisterHandlers registers all the recognizable handlers
func RegisterHandlers(f func(
    name string,
    handler func(
        context.Context,
        map[string]interface{},
    ) (http.Handler, error),
)) {
    f("custom-middleware", CustomMiddlewareHandler)
}

// CustomMiddlewareHandler creates a custom middleware
func CustomMiddlewareHandler(
    ctx context.Context,
    extra map[string]interface{},
) (http.Handler, error) {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Custom logic here
        fmt.Println("Custom middleware executed")
        
        // Add custom header
        w.Header().Set("X-Custom-Middleware", "active")
        
        // Continue to next handler
        next := ctx.Value("next").(http.Handler)
        next.ServeHTTP(w, r)
    }), nil
}

func main() {
    // Plugin initialization
}
```

### Plugin Configuration

```json
{
  "extra_config": {
    "plugin/http-server": {
      "name": ["custom-middleware"],
      "custom-middleware": {
        "enabled": true,
        "config": {
          "param1": "value1",
          "param2": "value2"
        }
      }
    }
  }
}
```

## Configuration Templates

### Flexible Configuration with Templates

```json
{
  "version": 3,
  "name": "{{ .service_name }}",
  "port": {{ .port }},
  "host": ["{{ .host }}"],
  "timeout": "{{ .timeout }}",
  "endpoints": [
    {{ range .endpoints }}
    {
      "endpoint": "{{ .endpoint }}",
      "method": "{{ .method }}",
      "backend": [
        {
          "url_pattern": "{{ .backend.url }}",
          "host": ["{{ .backend.host }}"],
          "method": "{{ .backend.method | default .method }}"
        }
      ]
    }{{ if not .last }},{{ end }}
    {{ end }}
  ]
}
```

### Environment Variables

```json
{
  "version": 3,
  "name": "{{ env "SERVICE_NAME" }}",
  "port": {{ env "PORT" | default "8080" }},
  "host": ["{{ env "HOST" | default "http://localhost:8080" }}"],
  "extra_config": {
    "telemetry/logging": {
      "level": "{{ env "LOG_LEVEL" | default "INFO" }}"
    }
  }
}
```

### Partial Templates

```json
// backends.tmpl
{
  "url_pattern": "{{ .url_pattern }}",
  "encoding": "{{ .encoding | default "json" }}",
  "method": "{{ .method | default "GET" }}",
  "host": ["{{ .host }}"]
  {{ if .extra_config }}
  ,"extra_config": {{ marshal .extra_config }}
  {{ end }}
}

// main configuration
{
  "endpoints": [
    {
      "endpoint": "/users/{id}",
      "backend": [
        {{ template "backends.tmpl" .user_service }}
      ]
    }
  ]
}
```

## Performance Optimization

### High-Performance Configuration

```json
{
  "version": 3,
  "name": "High-Performance Gateway",
  "port": 8080,
  "read_timeout": "5s",
  "write_timeout": "5s",
  "idle_timeout": "60s",
  "read_header_timeout": "5s",
  "dialer_timeout": "5s",
  "dialer_keep_alive": "60s",
  "dialer_fallback_delay": "300ms",
  "disable_keep_alives": false,
  "disable_compression": false,
  "max_idle_connections": 100,
  "max_idle_connections_per_host": 10,
  "response_header_timeout": "5s",
  "expect_continue_timeout": "1s",
  "output_encoding": "json",
  "cache_ttl": "3600s",
  "timeout": "2000ms"
}
```

### Connection Pooling

```json
{
  "backend": [
    {
      "url_pattern": "/api/users",
      "host": ["http://user-service:8001"],
      "extra_config": {
        "backend/http": {
          "return_error_details": "backend_alias"
        },
        "backend/http/client": {
          "max_idle_connections": 100,
          "max_idle_connections_per_host": 10,
          "timeout": "5s",
          "dialer_timeout": "2s",
          "dialer_keep_alive": "30s",
          "disable_compression": false
        }
      }
    }
  ]
}
```

## Testing and Debugging

### Debug Mode

```bash
# Start with debug mode
krakend run --config krakend.json --debug

# Environment variable
export KRAKEND_DEBUG=1
krakend run --config krakend.json
```

### Configuration Validation

```bash
# Check configuration syntax
krakend check --config krakend.json

# Detailed validation
krakend validate --config krakend.json

# Configuration audit
krakend audit --config krakend.json
```

### Testing Endpoints

```bash
# Test basic endpoint
curl -X GET "http://localhost:8080/v1/users/123"

# Test with headers
curl -X GET "http://localhost:8080/v1/users/123" \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json"

# Test POST endpoint
curl -X POST "http://localhost:8080/v1/users" \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'

# Test with query parameters
curl -X GET "http://localhost:8080/v1/users?limit=10&offset=0"

# Test health endpoint
curl -X GET "http://localhost:8080/__health"
```

## Production Deployment

### Docker Production Setup

```dockerfile
# Dockerfile
FROM devopsfaith/krakend:2.4
COPY krakend.json /etc/krakend/krakend.json
COPY templates/ /etc/krakend/templates/
EXPOSE 8080
ENTRYPOINT ["krakend"]
CMD ["run", "--config", "/etc/krakend/krakend.json"]
```

### Kubernetes Production Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krakend-production
  labels:
    app: krakend
    environment: production
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: krakend
  template:
    metadata:
      labels:
        app: krakend
    spec:
      containers:
      - name: krakend
        image: devopsfaith/krakend:2.4
        ports:
        - containerPort: 8080
        - containerPort: 8090  # Metrics
        command: ["krakend"]
        args: ["run", "--config", "/etc/krakend/krakend.json"]
        volumeMounts:
        - name: krakend-config
          mountPath: /etc/krakend
        env:
        - name: KRAKEND_PORT
          value: "8080"
        - name: GOMAXPROCS
          valueFrom:
            resourceFieldRef:
              resource: limits.cpu
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: krakend-config
        configMap:
          name: krakend-config
---
apiVersion: v1
kind: Service
metadata:
  name: krakend-service
  labels:
    app: krakend
spec:
  selector:
    app: krakend
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
    - name: metrics
      protocol: TCP
      port: 8090
      targetPort: 8090
  type: LoadBalancer
```

### Monitoring Stack

```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: krakend-metrics
spec:
  selector:
    matchLabels:
      app: krakend
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

---

*For more detailed information, visit the [official KrakenD documentation](https://www.krakend.io/docs/)*
