# Agent Day 2: Bakery and ShareSphere AI Assistant

This notebook demonstrates how to build an AI assistant with a system prompt, conversation memory, function logging, prompt comparisons, few-shot classification, structured JSON output, and memory-window limits.

## 1. Setup

The notebook installs the Groq and OpenAI Python SDKs:

```python
pip install -q groq openai
```

It loads the Groq API key from Google Colab Secrets and creates a Groq client. The model used is `openai/gpt-oss-20b`.

```python
os.environ["GROQ_API_KEY"] = userdata.get("Groq")
client = Groq()
MODEL = "openai/gpt-oss-20b"
```

The API key is kept in Colab Secrets instead of being written directly into the notebook.

## 2. Bakery Menu

The bakery menu is stored as a Python dictionary:

| Item | Price |
|---|---:|
| Plum cake | Rs 500 |
| Chocolate cake | Rs 650 |
| Cupcake | Rs 80 |

```python
MENU = {
    "plum cake": 500,
    "chocolate cake": 650,
    "cupcake": 80
}
```

The menu is converted to JSON and inserted into the system prompt. This is a prompt-injection-style setup, but in this notebook it is used only to give the model reliable facts for answering customers.

## 3. System Prompt

The `SYSTEM` prompt defines the assistant's role and behavior:

- Role: Assistant for Sweet Crumbs Home Bakery
- Fact: Menu prices
- Fact: Orders require 24 hours' notice
- Fact: Eggless versions are available
- Rule: Do not promise a delivery time
- Rule: Ask the owner when information is unavailable
- Rule: Remember the customer's name and order
- Format: Reply in under 50 words and match English or Telugu

This design helps keep the answers short and grounded in the provided facts.

## 4. Price Calculation and Logging

A `log_call` decorator logs every call made to a decorated function:

```python
@log_call
def get_price(item: str, qty: int = 1) -> int:
    return MENU[item.lower().strip()] * qty
```

For example:

```text
get_price("cupcake", 6) = 480
```

The decorator logs the function name, arguments, and return value:

```text
[log] get_price('cupcake', 6) -> 480
```

This is useful for debugging and tracking function calls.

## 5. Conversation History

The `history` list stores:

- The system prompt
- Customer messages
- Assistant replies

```python
history = [{"role": "system", "content": SYSTEM}]
```

The `chat()` function:

1. Adds the user's message to `history`.
2. Keeps the system prompt and a limited number of recent messages.
3. Sends the recent history to the Groq model.
4. Stores the assistant's reply in `history`.

```python
recent = [history[0]] + history[1:][-MAX_TURNS:]
```

The default value `MAX_TURNS = 8` limits conversation memory to the system prompt plus the latest eight messages.

## 6. API Error Handling

The chat function handles common API problems:

- Rate limit: asks the customer to retry in a minute
- Connection problem: asks the customer to retry shortly

This prevents the notebook from stopping due to temporary API failures.

## 7. Prompt Rules Experiment

The notebook compares two system prompts:

- **Without RULES:** It states directly that a one-hour delivery is not possible.
- **With RULES:** It does not make a delivery claim and asks the owner to confirm.

The recorded responses were:

```text
WITHOUT RULES:
Hi! Thanks for reaching out. We need a 24-hour notice for orders, so we
can't deliver in 1 hour. All cakes are eggless. Let us know if you’d like
to place an order for a later date. Cheers!

WITH RULES:
I’m not sure about a 1-hour delivery. I’ll check with the owner and get
back to you shortly. Thanks for reaching out!
```

This shows that explicit rules can change the model from a direct answer to a safer, uncertainty-aware response.

## 8. Few-Shot Classification

A classification prompt teaches the model three examples:

- `"I want 2 plum cakes"` -> `ORDER`
- `"How much is a chocolate cake?"` -> `PRICE_INQUIRY`
- `"Can you deliver to Kakinada?"` -> `DELIVERY`

The prompt requires the model to reply only with the label.

The test messages were classified as:

| Customer message | Classification |
|---|---|
| What is the cost of 6 cupcakes? | PRICE_INQUIRY |
| Book 1 eggless chocolate cake for Friday. | ORDER |
| Do you offer home delivery? | DELIVERY |
| Price for plum cake please? | PRICE_INQUIRY |
| I'd like to order 3 cupcakes. | ORDER |

Using `temperature=0` makes the classifications more deterministic.

## 9. Structured JSON Output

The notebook asks the model to convert Ravi's order into JSON:

```json
{
  "name": "Ravi",
  "item": "chocolate cake",
  "qty": 2,
  "date": "tomorrow",
  "eggless": true
}
```

The result is parsed with `json.loads()` and converted into a Python dictionary:

```python
order = json.loads(raw.strip().strip("`").removeprefix("json").strip())
```

If the model returns invalid JSON, `JSONDecodeError` is caught and displayed.

## 10. Conversation Memory Limit

The notebook reduces `MAX_TURNS` to `2`:

```python
MAX_TURNS = 2
history = [{"role": "system", "content": SYSTEM}]
```

This keeps only the latest one user message and one assistant reply after the system prompt.

As the conversation continues, Ravi's name is removed from the history. The final result was:

```text
What is my name?
I don’t have your name yet. Could you please tell me?
```

This demonstrates that increasing or decreasing `MAX_TURNS` directly affects what the assistant can remember.

## 11. ShareSphere Student Freelancing Assistant

The final example creates a second business assistant named ShareSphere.

| Service | Price |
|---|---:|
| Web development | Rs 800 |
| UI/UX | Rs 500 |
| PPT | Rs 250 |
| Data analytics | Rs 700 |

The system prompt includes these rules:

- Only verified college students can register
- Payouts are processed through UPI within 24 hours
- Never share student phone numbers
- Never promise exact delivery times
- Ask the team for unavailable information
- Remember the customer's name and order

The decorated service-price function calculates a UI/UX order correctly:

```text
get_service_price("ppt", 2) -> 500
Total for 2 PPTs: 500
```

For 2 UI/UX tasks, each costing Rs 500, the chatbot calculated Rs 1,000.

When asked for a designer's phone number, it refused to share the number and offered to connect the student through the platform. It later correctly remembered:

```text
Name: Sharif
Order: 2 UI/UX tasks
```

## Main Concepts Demonstrated

1. Loading API keys securely from Colab Secrets
2. Using a system prompt to define role and rules
3. Injecting structured business facts into a prompt
4. Using Python decorators for logging
5. Maintaining conversation history
6. Limiting memory to control context size
7. Comparing responses with and without prompt rules
8. Performing few-shot message classification
9. Generating and parsing JSON
10. Handling temporary API errors
