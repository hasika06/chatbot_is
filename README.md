# Customer Service Chatbot

A rule-based NLP chatbot built using **AIML (Artificial Intelligence Markup Language)** that answers common customer queries with intent detection, session management, and conversation logging.

---

## Project Structure

```
chatbot/
├── data/
│   └── customer_service.aiml   # Knowledge base (patterns + responses)
├── src/
│   └── chatbot.py              # Main chatbot class and CLI runner
├── tests/
│   └── test_chatbot.py         # Unit tests (preprocessing, intent, responses)
├── requirements.txt
└── README.md
```

---

## How It Works

### 1. AIML Pattern Matching
AIML defines `<category>` blocks, each with:
- `<pattern>` — what the user says (supports wildcards `*`)
- `<template>` — what the bot replies
- `<srai>` — redirect to another pattern (reduces duplication)

```xml
<category>
    <pattern>WHERE IS MY ORDER</pattern>
    <template>To track your order, visit "My Orders" in your account...</template>
</category>

<category>
    <pattern>TRACK MY ORDER</pattern>
    <template><srai>WHERE IS MY ORDER</srai></template>
</category>
```

### 2. NLP Preprocessing Pipeline
Before matching, user input goes through:
1. **Uppercasing** — AIML patterns are uppercase
2. **Contraction expansion** — "I'm" → "I AM", "can't" → "CANNOT"
3. **Punctuation removal** — strips `!`, `?`, `.` etc.
4. **Whitespace normalization**

### 3. Intent Detection
Alongside AIML, a keyword-scoring system classifies queries into 12 intent categories:
- `order_tracking`, `returns_refunds`, `shipping`, `payment`
- `account`, `product`, `subscription`, `discount`
- `complaint`, `greeting`, `farewell`, `escalation`

### 4. Session Management
Each user gets a session ID so the bot can maintain context across turns using AIML predicates.

---

## Supported Topics

| Topic | Example Query |
|-------|--------------|
| Order tracking | "Where is my order?", "Track my order" |
| Returns & refunds | "Return policy", "Where is my refund?" |
| Shipping | "Shipping options", "Do you ship internationally?" |
| Payment | "Payment methods", "My payment failed" |
| Account | "Forgot password", "Delete my account" |
| Products | "Is this in stock?", "Product warranty" |
| Subscriptions | "Cancel subscription", "Subscription plans" |
| Discounts | "Promo code", "Do you have any deals?" |
| Escalation | "Talk to a human", "Speak to an agent" |

---

## Setup & Installation

### Prerequisites
- Python 3.7+

### Install dependencies
```bash
pip install -r requirements.txt
```

### Run the chatbot
```bash
python src/chatbot.py
```

### Run unit tests
```bash
python -m pytest tests/ -v
```

---

## Sample Interaction

```
You: hello
Bot [greeting]: Hello! Welcome to our Customer Support. How can I help you today?

You: where is my order
Bot [order_tracking]: To track your order, please visit the "My Orders" section...

You: return policy
Bot [returns_refunds]: Our return policy:
  - Returns accepted within 30 days of delivery.
  - Items must be unused and in original packaging...

You: bye
Bot [farewell]: Goodbye! Thank you for contacting our support. Have a wonderful day!
```

---

## Key Concepts Demonstrated

- **AIML 2.0** — XML-based pattern-template dialogue system
- **Rule-based NLP** — deterministic pattern matching with wildcards
- **Preprocessing** — normalization pipeline for robust matching
- **Intent classification** — keyword scoring across 12 categories
- **`<srai>` tags** — symbolic reduction to avoid response duplication
- **Session predicates** — stateful conversation management
- **Unit testing** — pytest-based test coverage

---

## Extending the Bot

To add new topics:
1. Open `data/customer_service.aiml`
2. Add a new `<category>` block with your pattern and template
3. Add relevant keywords to the `intent_keywords` dict in `chatbot.py`

---

## Technologies Used

| Component | Technology |
|-----------|-----------|
| Dialogue Engine | AIML 1.0 / 2.0 |
| Language | Python 3 |
| NLP | Rule-based (keyword scoring + pattern matching) |
| Testing | unittest / pytest |
