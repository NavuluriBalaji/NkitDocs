# Deployment

Guide to deploying nkit applications to production.

## Local Development

### Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # or: venv\Scripts\activate on Windows

# Install nkit
pip install nkit openai

# Set API keys
export OPENAI_API_KEY="sk-..."
```

### Running Agents

```python
import asyncio
from nkit import Agent

async def main():
    agent = Agent(name="ProductionBot")
    result = await agent.aprocess([], "Your task")
    return result

result = asyncio.run(main())
```

## Docker Deployment

### Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Copy requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Set environment variables
ENV OPENAI_API_KEY=${OPENAI_API_KEY}
ENV PYTHONUNBUFFERED=1

# Run application
CMD ["python", "main.py"]
```

### requirements.txt

```
nkit==0.1.0
openai==1.0.0
python-dotenv==1.0.0
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  nkit-agent:
    build: .
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      LOG_LEVEL: info
    volumes:
      - ./logs:/app/logs
      - ./memory:/app/memory
    ports:
      - "8000:8000"
```

### Build and Run

```bash
# Build image
docker build -t nkit-agent:latest .

# Run container
docker run -e OPENAI_API_KEY="sk-..." nkit-agent:latest

# Using docker-compose
docker-compose up -d
```

## AWS Deployment

### Lambda Function

```python
# handler.py
from nkit import Agent
import json
import asyncio

agent = Agent(
    name="LambdaAgent",
    enable_memory=False  # Stateless
)

def lambda_handler(event, context):
    """AWS Lambda handler"""
    
    task = event.get('task', '')
    
    try:
        # Use sync version for Lambda
        result = agent.process([], task)
        
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

**Deployment:**

```bash
# Create deployment package
zip lambda_function.zip handler.py

# Deploy
aws lambda create-function \
  --function-name nkit-agent \
  --runtime python3.10 \
  --role arn:aws:iam::ACCOUNT:role/lambda-role \
  --handler handler.lambda_handler \
  --zip-file fileb://lambda_function.zip \
  --environment Variables={OPENAI_API_KEY=sk-...}
```

### Fargate Container

```bash
# Create ECR repository
aws ecr create-repository --repository-name nkit-agent

# Build and push
docker build -t nkit-agent .
docker tag nkit-agent:latest ACCOUNT.dkr.ecr.REGION.amazonaws.com/nkit-agent:latest
docker push ACCOUNT.dkr.ecr.REGION.amazonaws.com/nkit-agent:latest

# Deploy to Fargate via CloudFormation or console
```

## Google Cloud Deployment

### Cloud Run

```yaml
# app.yaml
runtime: python310

entrypoint: gunicorn -b :$PORT main:app

env:
  - name: OPENAI_API_KEY
    value: ${OPENAI_API_KEY}
```

```python
# main.py
from flask import Flask, request, jsonify
from nkit import Agent
import asyncio

app = Flask(__name__)
agent = Agent(name="CloudRunAgent")

@app.route('/process', methods=['POST'])
def process():
    data = request.json
    task = data.get('task')
    
    result = agent.process([], task)
    return jsonify(result)

if __name__ == '__main__':
    app.run(port=8080)
```

**Deploy:**

```bash
# Install Cloud SDK
gcloud init

# Deploy
gcloud run deploy nkit-agent \
  --source . \
  --platform managed \
  --region us-central1 \
  --set-env-vars OPENAI_API_KEY=sk-...
```

## Azure Deployment

### Function App

```python
# function_app.py
import azure.functions as func
from nkit import Agent

agent = Agent(name="AzureAgent")

def main(req: func.HttpRequest) -> func.HttpResponse:
    task = req.params.get('task')
    
    try:
        result = agent.process([], task)
        return func.HttpResponse(
            json.dumps(result),
            status_code=200
        )
    except Exception as e:
        return func.HttpResponse(
            json.dumps({'error': str(e)}),
            status_code=500
        )
```

**Deploy:**

```bash
# Create function app
func azure functionapp publish nkit-agent-app

# Or via Azure CLI
az functionapp create \
  --resource-group mygroup \
  --consumption-plan-location westus \
  --runtime python \
  --runtime-version 3.10 \
  --name nkit-agent-app
```

## Kubernetes Deployment

### Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nkit-agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nkit-agent
  template:
    metadata:
      labels:
        app: nkit-agent
    spec:
      containers:
      - name: agent
        image: myregistry.azurecr.io/nkit-agent:latest
        ports:
        - containerPort: 8000
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-secret
              key: api-key
        - name: LOG_LEVEL
          value: "info"
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
---
apiVersion: v1
kind: Service
metadata:
  name: nkit-agent-service
spec:
  selector:
    app: nkit-agent
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

**Deploy:**

```bash
# Create secret
kubectl create secret generic openai-secret \
  --from-literal=api-key=sk-...

# Deploy
kubectl apply -f deployment.yaml

# Check status
kubectl get pods
kubectl get service nkit-agent-service
```

## Vercel Deployment

### Next.js API Route

```python
# pages/api/agent.py
from nkit import Agent
import json

agent = Agent(name="VercelAgent")

def handler(request):
    """Vercel serverless function"""
    
    if request.method != 'POST':
        return {
            'statusCode': 405,
            'body': json.dumps({'error': 'Method not allowed'})
        }
    
    try:
        body = json.loads(request.body)
        task = body.get('task')
        
        result = agent.process([], task)
        
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

**Deploy:**

```bash
# Create vercel.json
{
  "functions": {
    "pages/api/agent.py": {
      "runtime": "python3.10"
    }
  }
}

# Deploy
vercel deploy
```

## Production Best Practices

### Environment Management

```python
import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env file

agent = Agent(
    name="ProductionAgent",
    model=os.getenv("LLM_MODEL", "gpt-3.5-turbo"),
    temperature=float(os.getenv("TEMPERATURE", "0.3"))
)
```

**.env file:**
```
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-3.5-turbo
TEMPERATURE=0.3
LOG_LEVEL=info
MAX_ITERATIONS=10
```

### Error Handling

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

try:
    result = agent.process([], "task")
    logger.info(f"Success: {result['status']}")
except TimeoutError:
    logger.error("Agent timeout")
    # Fallback logic
except Exception as e:
    logger.error(f"Agent error: {e}")
    # Send alert
```

### Monitoring

```python
import time
from datetime import datetime

@agent.hook("on_process_complete")
def log_metrics(result):
    logger.info({
        "timestamp": datetime.now().isoformat(),
        "status": result["status"],
        "iterations": result["iterations"],
        "tokens": result["tokens_used"]["total"],
        "duration": result.get("duration", 0)
    })
```

### Resource Limits

```python
agent = Agent(
    name="ProductionBot",
    max_iterations=10,      # Prevent infinite loops
    timeout=300,            # 5 minute limit
    enable_memory=True      # But prune periodically
)

# Periodic cleanup
async def cleanup():
    while True:
        await asyncio.sleep(3600)
        agent.memory.prune(keep_last=20)
```

### Security

```python
# Use environment variables for secrets
import os

API_KEY = os.getenv("OPENAI_API_KEY")
if not API_KEY:
    raise ValueError("OPENAI_API_KEY not set")

# Validate inputs
def validate_input(user_input: str) -> bool:
    if len(user_input) > 10000:
        return False  # Too long
    if any(keyword in user_input.lower() for keyword in ["delete", "drop"]):
        return False  # Dangerous
    return True

# Sanitize logs (don't log API keys)
def safe_log(data):
    data_copy = data.copy()
    data_copy.pop("api_key", None)
    logger.info(data_copy)
```

## Scaling

### Horizontal Scaling

```bash
# Docker Compose with multiple instances
docker-compose up -d --scale agent=5

# Kubernetes autoscaling
kubectl autoscale deployment nkit-agent \
  --min=3 --max=10 --cpu-percent=80
```

### Load Balancing

```yaml
# With nginx
upstream nkit_agents {
    server agent1:8000;
    server agent2:8000;
    server agent3:8000;
}

server {
    location /process {
        proxy_pass http://nkit_agents;
    }
}
```

## Monitoring

### Health Checks

```python
@app.route('/health', methods=['GET'])
def health():
    return {'status': 'healthy'}, 200

# Kubernetes health check
livenessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 30
  periodSeconds: 10
```

### Logging

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.StreamHandler(),  # stdout
        logging.FileHandler('/logs/agent.log')  # file
    ]
)
```

## Troubleshooting

### Memory Leaks

```python
# Monitor memory usage
import tracemalloc
tracemalloc.start()

# Periodically log
current, peak = tracemalloc.get_traced_memory()
logger.info(f"Memory: {current / 1024 / 1024:.1f}MB")
```

### Slow Startup

```python
# Pre-load models
from nkit.llms import OpenAIAdapter
llm = OpenAIAdapter()  # Initialize on startup
agent = Agent(llm=llm)
```

## Next Steps

- [Architecture](../core-concepts/architecture.md)
- [Performance Tuning](../advanced/performance.md)
- [Monitoring](../advanced/monitoring.md)
