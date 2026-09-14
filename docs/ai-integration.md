# Phase 3 — Adding AI to Your Microservices

Embed AI directly into the microservice code so your services can call Claude programmatically — using the same gateway you've been chatting with.

## How it works

The Claude AI gateway is already running on the cluster. Your microservice calls its `/run` endpoint — send a prompt, get a text response. No API keys in your code, no new infrastructure.

```
User request → your microservice → Claude gateway → Anthropic API → response
```

---

## Step 1 — Start the microservices

Open a terminal in Dev Spaces and get the services running:

```bash
pip install -r requirements.txt
./start-local.sh
```

Services will be available at:

| Service | URL |
|---|---|
| Gateway | http://localhost:8000 |
| Users | http://localhost:8001 |
| Products | http://localhost:8002 |
| Inventory | http://localhost:8003 |
| Orders | http://localhost:8004 |

Dev Spaces exposes these ports automatically — check the Ports panel in VS Code if the browser doesn't open.

---

## Step 2 — Test the gateway from the terminal

Before touching any service code, confirm the gateway responds:

```bash
TOKEN=$(cat ~/.claude-token)

curl -s -X POST http://claude-gateway.claude-sandbox.svc:8080/run \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Suggest 3 products for someone who ordered a laptop and a mouse"}'
```

You should see a text response stream back. Once this works, the same call from Python code is identical.

---

## Step 3 — Add the helper to a service

Add this to the service you want to enhance (e.g. `orders-service/main.py`):

```python
import httpx
import os

GATEWAY_URL = os.environ.get(
    "CLAUDE_GATEWAY_URL",
    "http://claude-gateway.claude-sandbox.svc:8080"
)

def _load_token() -> str:
    return open(os.path.expanduser("~/.claude-token")).read().strip()

def ask_claude(prompt: str) -> str:
    resp = httpx.post(
        f"{GATEWAY_URL}/run",
        headers={"Authorization": f"Bearer {_load_token()}"},
        json={"prompt": prompt},
        timeout=120,
    )
    resp.raise_for_status()
    return resp.text
```

---

## Step 4 — Add an AI-powered endpoint

Use `ask_claude()` inside a new route. Example — product recommendations based on order history:

```python
@app.get("/orders/{user_id}/recommend")
async def recommend(user_id: str, db: Session = Depends(get_db)):
    orders = db.query(Order).filter(Order.user_id == user_id).all()
    order_summary = [{"product": o.product_name, "quantity": o.quantity} for o in orders]

    prompt = f"""
    A customer has the following order history: {order_summary}
    Suggest 3 products they might like next. One line per product.
    """

    return {"user_id": user_id, "recommendations": ask_claude(prompt)}
```

---

## Step 5 — Test the new endpoint

```bash
# First create a user and some orders using the demo script
./demo.sh

# Then call your new endpoint
curl http://localhost:8000/orders/<user-id>/recommend
```

---

## Step 6 — Commit and push via the agent

Switch to the Claude chat and ask the agent to commit your changes:

```
Commit my changes with a clear message and push to GitLab
```

Then open an MR:

```
Open a draft MR for my AI integration changes
```

---

## Other ideas to try

| Endpoint | Service | What to ask Claude |
|---|---|---|
| `GET /products/{id}/description` | products | Generate a rich marketing description from name + price |
| `POST /orders/validate` | orders | Is this order suspicious? Flag potential fraud |
| `GET /inventory/reorder-suggestion` | inventory | How much should we reorder based on current stock? |
| `GET /users/{id}/summary` | users | Summarise this user's activity in plain English |
