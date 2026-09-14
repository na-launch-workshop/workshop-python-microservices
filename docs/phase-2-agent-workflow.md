# Phase 2 — Working with the Microservices Using the AI Agent

In this phase you use the Claude AI agent to explore, modify, and commit changes to the microservices codebase — all from the Dev Spaces terminal, without leaving your workspace.

## Start the agent

Open a terminal in Dev Spaces and run:

```bash
python3.11 claude-client/client.py chat
```

The agent will:
1. Register you automatically
2. Clone your fork of this repo into its sandbox
3. Show you the files
4. Drop you into an interactive prompt

## What to try

**Explore the codebase:**
```
What does the orders service do and how does it communicate with the other services?
```

**Make a code change:**
```
Add a /health endpoint to the gateway service that checks all downstream services are reachable and returns 503 if any are down
```

**Test the change:**
```
Run the gateway service and test the /health endpoint
```

**Commit and push:**
```
Commit the changes with a clear message and push to GitLab
```

**Open a Merge Request:**
```
Open a draft MR for my changes
```

## The workflow

```
You (chat prompt)
      ↓
Claude agent (Kata sandbox)
  → reads your code
  → makes changes
  → runs tests
  → git commit
  → git push → session branch (ai/<you>/<timestamp>)
      ↓
GitLab MR (you review the diff)
      ↓
Dev Spaces (git fetch + checkout to test locally)
```

The agent always works on its own branch — it never touches `main`. You review the MR before anything merges.

## Tips

- Be specific about what you want changed and why
- Ask the agent to run the code and show you the output before committing
- If something goes wrong, ask it to explain what it tried and why it failed
- Type `/reset` to clear the conversation and start fresh
- Press Enter twice to submit a multi-line prompt
