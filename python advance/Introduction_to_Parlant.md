# Introduction to Parlant

## 1. What is Parlant?

**Parlant** is a Python-based SDK designed to build **LLM-powered agents** with structured behavior, tool usage, and contextual memory.  
It provides a clean abstraction for creating **agentic AI systems** without requiring deep orchestration frameworks.

Parlant focuses on:
- Agent behavior control
- Tool calling
- Context and memory handling
- Built-in server and playground UI

> **Parlant helps you create intelligent AI agents that can think, act, remember, and interact through tools.**

---

## 2. Why Parlant?

Modern Generative AI systems are moving from **chatbots** to **agents** that:
- Perform actions
- Use tools
- Follow rules
- Maintain memory
- Run as services

Parlant addresses this shift by offering:
- Minimal setup
- Opinionated architecture
- Async-first design
- Natural language behavior control

---

## 3. Installing Parlant

```bash
pip install parlant
```

### Recommended setup

```bash
python -m venv venv
source venv/bin/activate
pip install parlant
```

---

## 4. Core Components

### Server
Hosts agents and provides a playground UI.

### Agent
Represents an AI entity with role and behavior.

### Tools
Functions the agent can call.

### Context & Memory
Variables that persist across interactions.

### Guidelines
Natural-language rules controlling behavior.

---

## 5. How It Works

1. User sends message  
2. Agent evaluates guidelines  
3. Tools are executed  
4. Context is updated  
5. Agent responds  

---

## 6. Playground

Access at:
```
http://localhost:8000
```

---

## 7. Summary

Parlant is ideal for:
- Teaching Agentic AI
- Rapid prototyping
- AI service deployment
