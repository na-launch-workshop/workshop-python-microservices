# Phase 3 — Adding AI to Your Microservices

Use the Claude agent to write a new AI-powered feature directly into the microservices code. By the end of this phase you'll have a working endpoint that generates AI content from live database data.

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

## Step 2 — Seed some data

Run the demo script to create users, products, inventory and a test order:

```bash
./demo.sh
```

Keep the product ID printed by the script — you'll need it to test your new endpoint.

---

## Step 3 — Open the Claude agent

Click **Terminal** → **Run Task** → **Start Claude AI**.

> Or from the terminal: `python3.11 claude-client/client.py chat`

---

## Step 4 — Ask the agent to build the feature

Paste this prompt exactly into the chat and press Enter twice:

```
Clone my GitLab repo workshop-python-microservices.

Add a new endpoint GET /products/{product_id}/describe to the products service.
It should:
1. Fetch the product from the SQLite database by ID (return 404 if not found)
2. Call the Claude AI gateway at http://claude-gateway.claude-sandbox.svc:8080/run
   using the token from ~/.claude-token
3. Ask Claude to write a 2-3 sentence marketing description for the product
   based on its name and price
4. Return a JSON response with the product data plus an "ai_description" field

Also add httpx to the products-service requirements if it isn't there already.
Test it by starting the service and calling the endpoint.
Commit the working code.
```

Watch the agent read the existing code, implement the feature, run the service, test it, and commit — all without leaving the chat.

---

## Step 5 — Test the endpoint yourself

Once the agent confirms it's working:

```bash
# Replace <product-id> with the ID from demo.sh output
curl http://localhost:8002/products/<product-id>/describe
```

You should get back something like:

```json
{
  "id": "abc-123",
  "name": "Laptop",
  "price": 1299.99,
  "ai_description": "The Laptop is a high-performance computing solution perfect for professionals who demand reliability and speed. Priced competitively at $1,299.99, it offers exceptional value for developers and power users alike. Whether you're coding, designing, or multitasking, this machine delivers the performance you need."
}
```

---

## Step 6 — Push and open an MR

In the chat:

```
Push my changes and open a draft MR
```

The agent pushes to a session branch and returns the GitLab MR URL. Review the diff in GitLab, then pull the branch in Dev Spaces to test the full stack.

---

## Other features to try

Once you've done the exercise above, pick another:

| Endpoint | Service | Prompt idea |
|---|---|---|
| `GET /orders/{user_id}/recommend` | orders | Suggest products based on a user's order history |
| `POST /orders/validate` | orders | Detect suspicious or fraudulent orders |
| `GET /inventory/reorder-suggestion` | inventory | Suggest reorder quantities based on stock levels |
| `GET /users/{id}/summary` | users | Summarise a user's activity in plain English |
