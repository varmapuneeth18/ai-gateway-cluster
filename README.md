# AI Gateway Cluster

A distributed AI serving infrastructure with multiple model servers and a unified gateway.

## Project Structure

```
ai-gateway-cluster/
├── infra/              # Kubernetes YAML files & Lightning Work configurations
├── qwen_server/        # SGLang Qwen model serving
├── kimi_server/        # Kimi2.5 model serving
└── agent_gateway/      # Unified API gateway (Lightning/LitServe/FastAPI)
```

## Components

### infra/
Kubernetes deployment configurations and Lightning Work setup for orchestrating the distributed AI serving infrastructure.

### qwen_server/
Dockerfile and scripts to run SGLang with Qwen models. Provides high-performance inference for Qwen model family.

### kimi_server/
Dockerfile and scripts to run Kimi2.5 models. Dedicated serving infrastructure for Kimi2.5.

### agent_gateway/
Unified API gateway built with Lightning/LitServe/FastAPI. Routes requests to appropriate model servers and provides a unified interface.

## Getting Started

1. Set up infrastructure: `cd infra/`
2. Build and deploy model servers
3. Launch the agent gateway
