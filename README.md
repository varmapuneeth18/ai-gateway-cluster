# AI Gateway Cluster

Distributed AI model serving infrastructure with intelligent load balancing across multiple LLMs.

## Features

- **Multi-Model Routing:** Intelligent API gateway distributes requests across LLMs
- **Load Balancing:** Automatic failover and request distribution
- **Kubernetes Native:** Scalable container orchestration
- **Production Ready:** Built for high-availability AI applications

## Tech Stack

- **API Gateway:** FastAPI
- **Orchestration:** Kubernetes, Docker
- **Model Serving:** SGLang

## Deployment

```bash
# Build containers
docker build -t ai-gateway .

# Deploy to Kubernetes
kubectl apply -f deployment.yaml
```

## Project Structure

```
ai-gateway-cluster/
├── infra/              # Kubernetes YAML files & Lightning Work configurations
├── qwen_server/        # SGLang Qwen model serving
├── kimi_server/        # Kimi2.5 model serving
└── agent_gateway/      # Unified API gateway (Lightning/LitServe/FastAPI)
```

