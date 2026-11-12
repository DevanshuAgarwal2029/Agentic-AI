# FraudGuard AI - Backend API

Production-grade fraud detection API with ML predictions and AI Agent reasoning.

## Project Structure

```
backend/
├── app/
│   ├── main.py                 # FastAPI application entry point
│   ├── api/
│   │   └── routes.py           # Core API routes (transactions, predict)
│   ├── ai/                     # AI Agent module (NEW)
│   │   ├── agent/
│   │   │   ├── agent.py        # Fraud detection agent orchestration
│   │   │   └── routes.py       # Agent API endpoints
│   │   ├── rag/
│   │   │   ├── embedding.py    # Sentence embeddings + FAISS index
│   │   │   ├── retrieval.py    # Knowledge retrieval
│   │   │   └── knowledge_base.py # Fraud detection knowledge
│   │   └── llm/
│   │       └── llm_service.py  # LLM integration (OpenAI)
│   ├── db/
│   │   └── database.py         # SQLAlchemy database config
│   ├── models/
│   │   ├── transaction.py      # SQLAlchemy Transaction model
│   │   ├── fraud_model.pkl     # Trained RandomForest model
│   │   └── type_encoder.pkl    # Transaction type encoder
│   ├── schemas/
│   │   └── transaction.py      # Pydantic schemas
│   └── services/
│       ├── ml_service.py       # ML prediction service
│       └── data_service.py     # Database operations
├── scripts/
│   └── train_model.py          # Model training script
├── requirements.txt
└── README.md
```

## Setup

### 1. Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 2. Train Model (Optional - model already trained)

```bash
python scripts/train_model.py
```

### 3. Run Server

```bash
python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 4. Set OpenAI API Key (Optional)

For real LLM responses (instead of mock):

```bash
export OPENAI_API_KEY="sk-..."
```

## API Endpoints

### Core Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | API info |
| GET | `/api/health` | Health check |
| GET | `/api/transactions` | List transactions |
| GET | `/api/transaction/{id}` | Get single transaction |
| POST | `/api/predict` | ML-only fraud prediction |
| POST | `/api/load-data` | Load dataset into DB |
| GET | `/api/stats` | Dashboard statistics |

### AI Agent Endpoints (NEW)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/agent/analyze` | Full AI agent analysis (ML + RAG + LLM) |
| POST | `/api/agent/chat` | Chat with fraud detection assistant |
| GET | `/api/agent/knowledge-base` | Get all knowledge entries |
| POST | `/api/agent/knowledge-base/rebuild` | Rebuild FAISS index |

## AI Agent Pipeline

```
Transaction → ML Model → risk_score
                      ↓
                   RAG → Retrieved Knowledge
                      ↓
                   LLM → Reasoning
                      ↓
            Structured Decision Output
```

## Usage Examples

### 1. ML-Only Prediction

```bash
curl -X POST http://localhost:8000/api/predict \
  -H "Content-Type: application/json" \
  -d '{
    "step": 1,
    "type": "CASH_OUT",
    "amount": 993.84,
    "oldbalanceOrg": 1390.84,
    "newbalanceOrig": 397.0,
    "oldbalanceDest": 0.0,
    "newbalanceDest": 0.0
  }'
```

**Response:**
```json
{
  "risk_score": 0.0098,
  "prediction": 0,
  "is_high_risk": false
}
```

### 2. Full AI Agent Analysis (ML + RAG + LLM)

```bash
curl -X POST http://localhost:8000/api/agent/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "step": 1,
    "type": "TRANSFER",
    "amount": 181.0,
    "oldbalanceOrg": 181.0,
    "newbalanceOrig": 0.0,
    "oldbalanceDest": 0.0,
    "newbalanceDest": 0.0
  }'
```

**Response:**
```json
{
  "transaction_id": "N/A",
  "timestamp": "2026-04-05T12:00:00.000000",
  "ml_analysis": {
    "risk_score": 0.9999,
    "prediction": 1,
    "is_high_risk": true
  },
  "agent_decision": {
    "decision": "HIGH",
    "explanation": "Transaction assessed with ML risk score of 1.00. Based on fraud patterns indicating complete fund drainage...",
    "action": "BLOCK",
    "confidence": 1.0
  },
  "retrieved_knowledge": [
    {"id": "KB004", "category": "balance_pattern"},
    {"id": "KB002", "category": "transaction_type"},
    {"id": "KB009", "category": "risk_threshold"}
  ]
}
```

### 3. Chat with AI Agent

```bash
curl -X POST http://localhost:8000/api/agent/chat \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What transaction types have the highest fraud risk?"
  }'
```

**Response:**
```json
{
  "answer": "Based on the fraud detection knowledge base... TRANSFER and CASH_OUT transaction types have the highest fraud rates...",
  "sources": ["KB002", "KB007", "KB015"],
  "confidence": 0.85
}
```

## API Documentation

Once running, visit:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Model Performance

| Metric | Value |
|--------|-------|
| **Algorithm** | RandomForestClassifier |
| **ROC-AUC** | 0.9996 |
| **Fraud Recall** | 98% |
| **Features** | 6 (type, amount, balances) |

## Knowledge Base

The RAG system includes 15 fraud detection rules across categories:

| Category | Count |
|----------|-------|
| amount_risk | 1 |
| transaction_type | 3 |
| balance_pattern | 3 |
| temporal_pattern | 2 |
| risk_threshold | 2 |
| destination_pattern | 1 |
| behavioral_pattern | 1 |
| pattern_sequence | 2 |

## Transaction Types

| Type | Encoded | Fraud Risk |
|------|---------|------------|
| CASH_OUT | 0 | High |
| DEBIT | 1 | Low |
| PAYMENT | 2 | Low-Medium |
| TRANSFER | 3 | High |
| CASH_IN | 4 | Medium |

## CORS Configuration

Frontend allowed from:
- http://localhost:3000
- http://localhost:3001

## Future Extensions

- Multi-agent system with specialized agents
- Memory for past decision tracking
- Real ERP/banking integration
- Tool calling for external APIs
- Vector database upgrade (Pinecone, Weaviate)
- Fine-tuned LLM for fraud domain
