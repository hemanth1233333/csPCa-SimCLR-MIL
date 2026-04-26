# Production-Grade Offline RAG System for Research Papers

A complete, production-ready Retrieval-Augmented Generation (RAG) system designed for processing and querying research papers, running entirely offline with full Docker and Kubernetes support.

## 🚀 Features

### Core Capabilities
- **Offline Operation**: All models run locally, no internet required after setup
- **Research Paper Processing**: Advanced PDF parsing with metadata extraction
- **Intelligent Chunking**: Context-aware text splitting for optimal retrieval
- **Hybrid Search**: Combines semantic and keyword-based search
- **Contextual Retrieval**: Provides surrounding context for better answers
- **Conversation History**: Multi-turn conversations with context awareness
- **Document Summarization**: Automatic paper summarization

### Technical Stack
- **Vector Database**: Qdrant for fast similarity search
- **Embeddings**: sentence-transformers (all-MiniLM-L6-v2)
- **LLM**: Mistral-7B-Instruct (GGUF format via llama.cpp)
- **Caching**: Redis for query result caching
- **Metadata Storage**: PostgreSQL for document metadata
- **API Framework**: FastAPI with async support
- **Orchestration**: Docker Compose + Kubernetes
- **Monitoring**: Prometheus + Grafana

## 📋 Prerequisites

### Required
- Docker (20.10+)
- Docker Compose (2.0+)
- Python 3.11+
- 16GB+ RAM (32GB recommended)
- 50GB+ free disk space

### For Kubernetes
- kubectl (1.25+)
- Kubernetes cluster (minikube, k3s, or cloud provider)
- helm (3.0+)

## 🛠️ Installation

### Quick Start

1. **Clone and Setup**
```bash
git clone <repository-url>
cd rag-research-system
chmod +x deploy.sh
```

2. **Download LLM Model**
```bash
mkdir -p models
cd models
wget https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf
cd ..
```

3. **Run Deployment Script**
```bash
./deploy.sh
```

### Manual Docker Compose Deployment

1. **Build Services**
```bash
docker-compose build
```

2. **Start Services**
```bash
docker-compose up -d
```

3. **Verify Health**
```bash
curl http://localhost:8000/health
```

### Kubernetes Deployment

1. **Create Namespace**
```bash
kubectl apply -f infrastructure/kubernetes/namespace-config.yaml
```

2. **Deploy Storage & Databases**
```bash
kubectl apply -f infrastructure/kubernetes/storage-databases.yaml
```

3. **Wait for Databases**
```bash
kubectl wait --for=condition=ready pod -l app=postgres -n rag-system --timeout=300s
kubectl wait --for=condition=ready pod -l app=qdrant -n rag-system --timeout=300s
```

4. **Deploy Services**
```bash
kubectl apply -f infrastructure/kubernetes/services.yaml
```

5. **Check Status**
```bash
kubectl get pods -n rag-system
kubectl get services -n rag-system
```

## 📚 Usage

### Upload Research Papers

**Single File**
```bash
curl -X POST -F "file=@paper.pdf" http://localhost:8000/upload
```

**Bulk Upload**
```bash
for file in papers/*.pdf; do
    curl -X POST -F "file=@$file" http://localhost:8000/upload
    sleep 2
done
```

### Query the System

**Basic Query**
```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the main findings about transformer architecture?",
    "top_k": 5,
    "min_score": 0.7
  }'
```

**Response Format**
```json
{
  "query": "What are the main findings...",
  "answer": "Based on the research papers...",
  "sources": [
    {
      "text": "Excerpt from paper...",
      "score": 0.89,
      "document_id": "abc123",
      "filename": "attention_is_all_you_need.pdf",
      "title": "Attention Is All You Need"
    }
  ],
  "total_sources": 5,
  "processing_time": 2.34
}
```

### Hybrid Search
```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "neural network optimization",
    "use_hybrid": true,
    "keywords": ["adam", "learning rate", "gradient descent"],
    "top_k": 5
  }'
```

### Conversational Query
```bash
curl -X POST "http://localhost:8000/chat?query=What%20is%20BERT&conversation_id=user123&top_k=5"
```

### Document Summarization
```bash
curl -X POST "http://localhost:8000/summarize_document?document_id=abc123"
```

### Check Status
```bash
# Overall health
curl http://localhost:8000/health

# System statistics
curl http://localhost:8000/stats

# List documents
curl http://localhost:8001/documents
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     API Gateway                          │
│                   (FastAPI + CORS)                       │
└────────────┬────────────────────────────┬───────────────┘
             │                            │
    ┌────────▼────────┐          ┌───────▼────────┐
    │   Retrieval     │          │   Generation    │
    │    Service      │          │    Service      │
    │  (Embeddings +  │          │  (Mistral-7B)   │
    │   Search)       │          │                 │
    └────────┬────────┘          └─────────────────┘
             │
    ┌────────▼────────┐
    │   Ingestion     │
    │    Service      │
    │  (PDF Parser)   │
    └────────┬────────┘
             │
    ┌────────▼─────────────────────────────┐
    │  Storage Layer                       │
    ├──────────────┬────────────┬──────────┤
    │   Qdrant     │ PostgreSQL │  Redis   │
    │   (Vectors)  │ (Metadata) │ (Cache)  │
    └──────────────┴────────────┴──────────┘
```

## 🔧 Configuration

### Environment Variables

**Ingestion Service**
```bash
QDRANT_HOST=qdrant
QDRANT_PORT=6333
POSTGRES_HOST=postgres
POSTGRES_DB=rag_metadata
POSTGRES_USER=raguser
POSTGRES_PASSWORD=ragpass123
EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
```

**Generation Service**
```bash
MODEL_PATH=/app/models/mistral-7b-instruct-v0.2.Q4_K_M.gguf
CONTEXT_LENGTH=4096
GPU_LAYERS=0  # Set to >0 if GPU available
MAX_TOKENS=512
TEMPERATURE=0.7
```

### Resource Requirements

**Minimum** (Development)
- CPU: 4 cores
- RAM: 16GB
- Storage: 30GB

**Recommended** (Production)
- CPU: 8+ cores
- RAM: 32GB+
- Storage: 100GB+
- GPU: Optional (speeds up inference)

### Scaling

**Horizontal Scaling** (Kubernetes)
```bash
# Scale retrieval service
kubectl scale deployment retrieval -n rag-system --replicas=5

# Scale API gateway
kubectl scale deployment api-gateway -n rag-system --replicas=5
```

**Auto-scaling** is configured via HPA (Horizontal Pod Autoscaler) in Kubernetes manifests.

## 📊 Monitoring

### Access Dashboards

**Grafana**
- URL: http://localhost:3000
- Username: admin
- Password: admin

**Prometheus**
- URL: http://localhost:9090

**Qdrant Dashboard**
- URL: http://localhost:6333/dashboard

### Key Metrics

- Query latency (p50, p95, p99)
- Vector search performance
- LLM inference time
- Cache hit rate
- Document processing throughput
- API request rates

### Logs

**Docker Compose**
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api-gateway
docker-compose logs -f retrieval
```

**Kubernetes**
```bash
# All pods
kubectl logs -f -l app=api-gateway -n rag-system

# Specific pod
kubectl logs -f <pod-name> -n rag-system
```

## 🧪 Testing

### Unit Tests
```bash
pytest tests/unit/
```

### Integration Tests
```bash
pytest tests/integration/
```

### Load Testing
```bash
# Using locust
locust -f tests/load/locustfile.py --host=http://localhost:8000
```

### Test Query Examples
```bash
# Test upload
curl -X POST -F "file=@tests/fixtures/sample_paper.pdf" http://localhost:8000/upload

# Test query
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{"query": "test query", "top_k": 3}'

# Test health
curl http://localhost:8000/health
```

## 🔒 Security

### Best Practices
1. **Change Default Passwords**: Update PostgreSQL and Grafana passwords
2. **Network Isolation**: Use private networks in production
3. **API Authentication**: Add JWT or API key authentication
4. **Rate Limiting**: Implement rate limiting on API gateway
5. **HTTPS**: Use TLS/SSL in production

### Kubernetes Security
```yaml
# Add to deployments
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  capabilities:
    drop:
      - ALL
```

## 🚨 Troubleshooting

### Common Issues

**1. Out of Memory**
```bash
# Reduce batch sizes in ingestion
# Decrease CONTEXT_LENGTH in generation service
# Increase Docker memory limit
```

**2. Slow Query Response**
```bash
# Check Redis cache: redis-cli ping
# Monitor Qdrant: http://localhost:6333/metrics
# Check GPU utilization if available
```

**3. Model Not Loading**
```bash
# Verify model file exists
ls -lh models/

# Check service logs
docker-compose logs generation
```

**4. Database Connection Issues**
```bash
# Check database health
docker-compose ps
kubectl get pods -n rag-system

# Verify connection
docker-compose exec postgres psql -U raguser -d rag_metadata
```

### Debug Mode
```bash
# Enable debug logging
docker-compose up --build
# Check individual service logs
docker-compose logs -f <service-name>
```

## 📈 Performance Optimization

### Embedding Model Selection
- **Fast**: all-MiniLM-L6-v2 (default)
- **Balanced**: all-mpnet-base-v2
- **Best**: intfloat/e5-large-v2

### LLM Model Selection
- **Fast**: Mistral-7B Q4 (default)
- **Balanced**: Mistral-7B Q5
- **Best**: Mistral-7B Q8 or larger models

### Indexing Optimization
```python
# Adjust chunk size and overlap
CHUNK_SIZE = 512  # Smaller = more precise
CHUNK_OVERLAP = 50  # Larger = more context
```

### Caching Strategy
- Query results: 1 hour TTL
- Embeddings: Permanent
- Document metadata: Permanent

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 🙏 Acknowledgments

- Qdrant for vector database
- Hugging Face for embedding models
- llama.cpp for efficient LLM inference
- FastAPI framework
- The open-source community

## 📞 Support

- GitHub Issues: [Create an issue]
- Documentation: [Wiki]
- Email: support@example.com

## 🗺️ Roadmap

- [ ] Add support for more document formats (Word, HTML)
- [ ] Implement fine-tuning pipeline
- [ ] Add multi-language support
- [ ] GraphQL API
- [ ] Advanced analytics dashboard
- [ ] Mobile app integration
- [ ] Federated learning support
