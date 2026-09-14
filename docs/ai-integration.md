# Adding AI to Your Microservices

This guide covers Phase 3 of the workshop — embedding AI directly into the microservice code so your services can call Claude programmatically.

## How it works

The Claude AI gateway is already running on the cluster and holds the Anthropic API key. Your microservice calls the gateway's `/run` endpoint the same way the chat client does — send a prompt, get a text response back.

```
User request → your microservice → Claude gateway → Anthropic API → response
```

No API keys in your code. No new infrastructure. The gateway handles auth, rate limiting, and the API call.

## Prerequisites

- Your Dev Spaces workspace is open
- You have registered with the Claude gateway (the chat client does this automatically)
- Your token is stored at `~/.claude-token`

## The pattern

Add this helper to any service:

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
    """Send a prompt to Claude and return the response text."""
    resp = httpx.post(
        f"{GATEWAY_URL}/run",
        headers={"Authorization": f"Bearer {_load_token()}"},
        json={"prompt": prompt},
        timeout=120,
    )
    resp.raise_for_status()
    return resp.text
```

## Example — product recommendations

Add a `/recommend` endpoint to the orders service that suggests products based on a user's order history:

```python
# orders-service/main.py

@app.get("/orders/{user_id}/recommend")
async def recommend(user_id: str, db: Session = Depends(get_db)):
    # Fetch the user's order history from the database
    orders = db.query(Order).filter(Order.user_id == user_id).all()
    order_summary = [
        {"product": o.product_name, "quantity": o.quantity}
        for o in orders
    ]

    prompt = f"""
    A customer has the following order history:
    {order_summary}

    Suggest 3 products they might like next. Be concise — one line per product.
    """

    recommendations = ask_claude(prompt)
    return {"user_id": user_id, "recommendations": recommendations}
```

## Testing from Dev Spaces

Test the gateway call directly before wiring it into the service:

```bash
TOKEN=$(cat ~/.claude-token)

curl -s -X POST http://claude-gateway.claude-sandbox.svc:8080/run \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Suggest 3 products for someone who ordered a laptop and a mouse"}' \
  | cat
```

Once that works, the same call in Python code works identically.

## Other ideas

| Endpoint to add | Service | Prompt idea |
|---|---|---|
| `GET /products/{id}/description` | products | Generate a rich marketing description from name + price |
| `POST /orders/validate` | orders | Check if an order looks fraudulent |
| `GET /inventory/reorder-suggestion` | inventory | Suggest reorder quantities based on stock levels |
| `GET /users/{id}/summary` | users | Summarise a user's activity in plain English |

## Running in Dev Spaces vs locally

The gateway URL `http://claude-gateway.claude-sandbox.svc:8080` only resolves inside the cluster. If you run docker-compose locally on your laptop, set the env var to point at a port-forward or accept that AI features only work in Dev Spaces.

```bash
# In Dev Spaces — works automatically
CLAUDE_GATEWAY_URL=http://claude-gateway.claude-sandbox.svc:8080

# Local testing — use oc port-forward first
# oc port-forward svc/claude-gateway 8080:8080 -n claude-sandbox
CLAUDE_GATEWAY_URL=http://localhost:8080
```
