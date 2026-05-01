# ReceiptBuddy Gateway

Nginx API Gateway — reverse proxy routing to all microservices.

## Responsibilities

- Route requests to appropriate backend service
- CORS handling (server-level + location-level headers)
- Request size limits (20MB for receipt uploads)
- Single entry point on port 8000

## Routing

| Path | Upstream | Service |
|------|----------|---------|
| `/api/auth/` | `http://auth:8001` | Auth Service |
| `/api/receipts/`, `/api/expenses/`, etc. | `http://finance:8002` | Finance Service |
| `/api/employees/`, `/api/attendance/`, etc. | `http://hr:8003` | HR Service |
| `/api/ai/`, `/api/analytics/`, `/api/reports/` | `http://intelligence:8004` | Intelligence Service |
| `/api/storage/` | `http://minio:9000` | MinIO (image serving) |
| `/health` | — | Health check (returns 200) |

## CORS Strategy

- Server-level `add_header ... always` covers error responses (502, 404)
- Per-location `add_header` covers proxy responses
- `proxy_hide_header` strips upstream CORS to prevent `*,*` duplication
- `if ($request_method = OPTIONS) { return 204; }` handles preflight

## Quick Start

```bash
docker build -t receiptbuddy-gateway .
docker run -p 8000:8000 receiptbuddy-gateway
```

## Configuration

Edit `nginx.conf` to modify routing rules. Key settings:

```nginx
client_max_body_size 20M;        # Max upload size
proxy_connect_timeout 60s;       # Backend connection timeout
resolver 127.0.0.11 valid=10s;   # Docker DNS for service discovery
```

## Dependencies

- All microservices must be running for full functionality
- Docker DNS (127.0.0.11) for service name resolution
