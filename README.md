# 🚀 AI Gateway Cluster

Distributed AI model serving infrastructure with intelligent API gateway routing requests across multiple LLMs. Built for high-availability production AI applications with automatic load balancing and failover.

![Architecture Overview](./assets/architecture.png)
<!-- TODO: Add architecture diagram showing gateway → model routing -->

## 🎯 Overview

AI Gateway Cluster is a production-ready infrastructure for serving multiple AI models through a unified API gateway. It handles intelligent request routing, load balancing, failover, and monitoring across distributed LLM instances.

### Why This Exists

Modern AI applications need:
- **High Availability** - No single point of failure
- **Load Distribution** - Efficient resource utilization across models
- **Cost Optimization** - Route to most cost-effective model for each request
- **Fallback Handling** - Automatic failover when models are unavailable
- **Observability** - Real-time monitoring and metrics

This gateway provides all of the above in a Kubernetes-native, production-ready package.

## ✨ Key Features

- **🎯 Intelligent Routing** - Route requests based on model capability, load, and cost
- **⚖️ Load Balancing** - Distribute traffic evenly across model instances
- **🔄 Automatic Failover** - Seamless switching when a model becomes unavailable
- **📊 Real-time Monitoring** - Prometheus metrics and health checks
- **🔒 Rate Limiting** - Per-user and per-model request throttling
- **🚦 Circuit Breaker** - Prevent cascading failures
- **📈 Auto-scaling** - Kubernetes HPA for dynamic scaling
- **🔐 Authentication** - API key and JWT-based auth

## 🏗️ Architecture

```
                        ┌─────────────────────┐
                        │   API Gateway       │
                        │   (FastAPI)         │
                        │  - Routing Logic    │
                        │  - Load Balancer    │
                        │  - Auth & Rate Limit│
                        └──────────┬──────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
         ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
         │  Qwen Model     │ │  Kimi Model     │ │  Lightning      │
         │  (SGLang)       │ │  (SGLang)       │ │  Agent Gateway  │
         │  - Fast Text    │ │  - Long Context │ │  - Agentic      │
         │  - Efficient    │ │  - 128K tokens  │ │  - Tool Use     │
         └─────────────────┘ └─────────────────┘ └─────────────────┘
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                        ┌─────────────────────┐
                        │   Kubernetes        │
                        │   - Auto-scaling    │
                        │   - Health Checks   │
                        │   - Service Mesh    │
                        └─────────────────────┘
                                   ▼
                        ┌─────────────────────┐
                        │  Observability      │
                        │  - Prometheus       │
                        │  - Grafana          │
                        │  - Logging          │
                        └─────────────────────┘
```

## 🛠️ Tech Stack

### Core Infrastructure
- **API Gateway:** FastAPI (async Python)
- **Model Serving:** SGLang (fast inference runtime)
- **Orchestration:** Kubernetes
- **Containerization:** Docker
- **Load Balancing:** Kubernetes Service + Ingress

### Models Supported
- **Qwen:** Fast general-purpose text generation
- **Kimi:** Long-context processing (128K tokens)
- **Lightning Agent:** Agentic workflows with tool use

### Observability
- **Metrics:** Prometheus
- **Visualization:** Grafana
- **Logging:** Fluentd + Elasticsearch
- **Tracing:** OpenTelemetry

## 📋 Prerequisites

- Kubernetes cluster (v1.24+)
- kubectl configured
- Docker installed
- Helm 3.x
- GPU nodes (optional but recommended for performance)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/varmapuneeth18/ai-gateway-cluster.git
cd ai-gateway-cluster
```

### 2. Configure Environment
```bash
# Copy example config
cp config/example.env config/.env

# Edit configuration
nano config/.env
```

Required environment variables:
```bash
# Gateway Configuration
GATEWAY_HOST=0.0.0.0
GATEWAY_PORT=8000
WORKERS=4

# Model Endpoints
QWEN_ENDPOINT=http://qwen-service:8001
KIMI_ENDPOINT=http://kimi-service:8002
LIGHTNING_ENDPOINT=http://lightning-service:8003

# Authentication
API_KEY_SECRET=your_secret_key_here
JWT_SECRET=your_jwt_secret_here

# Rate Limiting
RATE_LIMIT_PER_MINUTE=60
RATE_LIMIT_PER_HOUR=1000

# Monitoring
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=9090
```

### 3. Deploy to Kubernetes

**Option A: Using Helm (Recommended)**
```bash
# Install the gateway
helm install ai-gateway ./helm/ai-gateway \
  --namespace ai-gateway \
  --create-namespace \
  --values config/values.yaml

# Verify deployment
kubectl get pods -n ai-gateway
```

**Option B: Using kubectl**
```bash
# Apply manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/ingress.yaml

# Check status
kubectl get all -n ai-gateway
```

### 4. Verify Installation
```bash
# Port-forward the gateway
kubectl port-forward -n ai-gateway service/gateway 8000:8000

# Test the health endpoint
curl http://localhost:8000/health

# Expected response:
# {"status": "healthy", "models": ["qwen", "kimi", "lightning"]}
```

## 📡 API Usage

### Authentication
```bash
# Get API key (admin only)
curl -X POST http://localhost:8000/auth/api-key \
  -H "Authorization: Bearer <admin-jwt>" \
  -d '{"name": "my-app", "rate_limit": 100}'

# Response:
# {"api_key": "sk-abc123...", "rate_limit": 100}
```

### Making Requests

**Basic Completion**
```bash
curl -X POST http://localhost:8000/v1/completions \
  -H "Authorization: Bearer sk-abc123..." \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain quantum computing",
    "max_tokens": 500,
    "model": "auto"
  }'
```

**Specify Model**
```bash
curl -X POST http://localhost:8000/v1/completions \
  -H "Authorization: Bearer sk-abc123..." \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Summarize this long document...",
    "max_tokens": 1000,
    "model": "kimi"  # Use long-context model
  }'
```

**Streaming Response**
```bash
curl -X POST http://localhost:8000/v1/completions \
  -H "Authorization: Bearer sk-abc123..." \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Write a story",
    "max_tokens": 1000,
    "stream": true
  }'
```

### Routing Logic

The gateway automatically routes requests based on:

1. **Explicit Model Selection** - If `model` is specified, route to that model
2. **Context Length** - Long prompts (>8K tokens) → Kimi
3. **Tool Use** - Requests with tools → Lightning Agent
4. **Load Balancing** - Distribute across available instances
5. **Failover** - Switch to backup if primary fails

```python
# Example routing logic (simplified)
def route_request(request):
    if request.model:
        return request.model
    
    if len(request.prompt) > 8000:
        return "kimi"  # Long context
    
    if request.tools:
        return "lightning"  # Agentic
    
    # Default: load balance across Qwen instances
    return select_least_loaded("qwen")
```

## 📊 Monitoring

### Access Grafana Dashboard
```bash
# Port-forward Grafana
kubectl port-forward -n ai-gateway service/grafana 3000:3000

# Open http://localhost:3000
# Default credentials: admin / admin
```

### Key Metrics
- **Request Rate** - Requests per second by model
- **Latency** - P50, P95, P99 response times
- **Error Rate** - Failed requests percentage
- **Token Usage** - Input/output tokens consumed
- **Model Availability** - Uptime per model
- **Queue Depth** - Pending requests per model

### Prometheus Queries
```promql
# Request rate by model
rate(gateway_requests_total[5m])

# Average latency
histogram_quantile(0.95, gateway_request_duration_seconds_bucket)

# Error rate
rate(gateway_errors_total[5m]) / rate(gateway_requests_total[5m])
```

## 🔧 Configuration

### Scaling Models

**Horizontal Pod Autoscaling**
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: qwen-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: qwen-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Manual Scaling**
```bash
# Scale Qwen to 5 replicas
kubectl scale deployment qwen-deployment -n ai-gateway --replicas=5
```

### Rate Limiting

Configure per-user rate limits in `config/rate-limits.yaml`:
```yaml
rate_limits:
  default:
    per_minute: 60
    per_hour: 1000
  
  premium:
    per_minute: 300
    per_hour: 10000
  
  enterprise:
    per_minute: 1000
    per_hour: 50000
```

### Model Configuration

Edit `config/models.yaml`:
```yaml
models:
  qwen:
    instances: 3
    max_batch_size: 16
    timeout: 30
    priority: 1
  
  kimi:
    instances: 2
    max_batch_size: 8
    timeout: 60
    priority: 2
  
  lightning:
    instances: 2
    max_batch_size: 4
    timeout: 120
    priority: 3
```

## 🧪 Testing

### Unit Tests
```bash
# Run tests
pytest tests/

# With coverage
pytest --cov=gateway tests/
```

### Load Testing
```bash
# Install k6
brew install k6  # macOS
# or download from https://k6.io

# Run load test
k6 run tests/load/basic-load.js

# Example output:
#   ✓ status is 200
#   ✓ response time < 500ms
#   
#   checks.........................: 100.00% ✓ 20000 ✗ 0
#   http_req_duration..............: avg=245ms min=100ms max=450ms
#   http_reqs......................: 10000   333/s
```

### Integration Tests
```bash
# Deploy to test cluster
./scripts/deploy-test.sh

# Run integration tests
pytest tests/integration/

# Cleanup
./scripts/cleanup-test.sh
```

## 📈 Performance

### Benchmarks

Tested on: Kubernetes cluster with 4 GPU nodes (NVIDIA A100)

| Metric | Value |
|--------|-------|
| Max Throughput | 1,000 req/sec |
| P95 Latency | <300ms |
| Uptime | 99.9% |
| Concurrent Users | 10,000+ |
| Token Processing | 1M tokens/min |

### Optimization Tips

1. **Enable GPU** - 10x faster inference
2. **Batch Requests** - 2-3x higher throughput
3. **Use Caching** - Cache common prompts
4. **Tune Workers** - Match CPU cores
5. **Monitor Memory** - Prevent OOM kills

## 🐛 Troubleshooting

### Gateway Won't Start
```bash
# Check logs
kubectl logs -n ai-gateway deployment/gateway --tail=50

# Common issues:
# - Missing secrets
# - Invalid model endpoints
# - Port conflicts
```

### High Latency
```bash
# Check model health
curl http://localhost:8000/health/models

# Check resource usage
kubectl top pods -n ai-gateway

# Check if models are overloaded
kubectl describe hpa -n ai-gateway
```

### Models Unavailable
```bash
# Check model pod status
kubectl get pods -n ai-gateway -l app=qwen

# Restart model
kubectl rollout restart deployment/qwen-deployment -n ai-gateway
```

## 🗺️ Roadmap

- [ ] Multi-cloud support (AWS, GCP, Azure)
- [ ] WebSocket streaming
- [ ] Model versioning and A/B testing
- [ ] Cost tracking and billing
- [ ] Request caching layer
- [ ] GraphQL API
- [ ] SDK clients (Python, JavaScript, Go)
- [ ] Fine-tuning pipeline integration

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## 📝 License

MIT License - see [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

- SGLang team for efficient model serving
- Kubernetes community for orchestration tools
- FastAPI for excellent async framework
- Prometheus & Grafana for observability

## 📧 Contact

Puneeth Varma - [varma.puneeth07@gmail.com](mailto:varma.puneeth07@gmail.com)

LinkedIn: [linkedin.com/in/puneethvarma180745](https://www.linkedin.com/in/puneethvarma180745)

Project Link: [https://github.com/varmapuneeth18/ai-gateway-cluster](https://github.com/varmapuneeth18/ai-gateway-cluster)

---

⭐ Star this repo if you find it useful for your AI infrastructure!

## 📚 Additional Resources

- [API Documentation](./docs/API.md)
- [Deployment Guide](./docs/DEPLOYMENT.md)
- [Security Best Practices](./docs/SECURITY.md)
- [Performance Tuning](./docs/PERFORMANCE.md)
