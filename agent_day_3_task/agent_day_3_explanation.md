# Agent Day 3 Notebook Explanation

This notebook, `agent_day_3.ipynb`, demonstrates how to build a small conversational AI assistant with LangChain, Groq, prompt engineering, testing, agents, and temperature settings in Google Colab.

## 1. Environment setup

The first code cell:

- Installs LangChain and the Groq LangChain integration.
- Imports `os`, Google Colab secret access, and `ChatGroq`.
- Reads a Groq API key named `Groq` from Colab Secrets.
- Initializes `openai/gpt-oss-20b` with a temperature of `0.3`.

Temperature `0.3` makes responses relatively focused and consistent.

## 2. Sweet Crumbs bakery assistant

The notebook defines a system prompt containing the bakery role, product facts, rules, and response format:

- Plum cake: Rs 500
- Chocolate cake: Rs 650
- Cupcake: Rs 80 each
- Orders require 24 hours' notice
- All cakes have eggless versions

The prompt instructs the model not to promise delivery details and to say `"I'll check with the owner"` for information outside the stated facts.

### LangChain pipeline

The chain uses LangChain Expression Language:

```text
ChatPromptTemplate -> ChatGroq -> StrOutputParser
```

The prompt contains:

- A system message with the bakery instructions
- `MessagesPlaceholder("history")` for previous conversation turns
- A human question placeholder

### Conversation memory

The `ask()` function sends the existing history to the chain and then appends both the human question and AI response to the history. This gives the model conversational memory.

The notebook demonstrates this memory by asking:

1. Whether eggless cakes are available
2. The user's name
3. Whether a cake can be ordered today, delivery to Yeleswaram, and the price of six cupcakes

The result correctly identified the user as Ravi, rejected same-day ordering because of the 24-hour rule, offered to check Yeleswaram delivery, and calculated `6 x Rs 80 = Rs 480`.

## 3. Automated keyword tests

The `run_tests()` function reads a CSV file containing questions and required keyword values.

For every test row, it:

1. Invokes the chain without conversation history.
2. Converts the answer and required keyword to lowercase.
3. Checks whether the keyword appears in the answer.
4. Prints `PASS` or `FAIL`.
5. Calculates and prints the pass rate.

This is a basic keyword-based evaluation method. It checks only whether expected text is present, not whether the complete answer is accurate or high quality.

The initial bakery test file produced **4/5 passes, or 80%**.

The failed case was same-day cake delivery. The response followed the 24-hour notice rule rather than claiming that the order was possible.

## 4. LangChain agent example

The notebook also creates an agent using `create_agent()`:

- The Groq model is used as the agent model.
- The tools list is empty.
- `SYSTEM` is supplied as the system prompt.

With no tools, the agent behaves similarly to the earlier chain. Its purpose here is to demonstrate the basic agent API and show that it can be extended with tools in a later implementation.

## 5. ShareSphere assistant

The notebook adapts the prompt for **ShareSphere**, a peer-to-peer freelancing platform for verified college students.

ShareSphere facts include:

- Web development: Rs 800
- UI/UX design: Rs 500
- PPT creation: Rs 250
- Data analytics: Rs 700
- Only verified college students can register
- Payouts are sent by UPI within 24 hours

The rules prevent the assistant from sharing personal contact details or promising exact delivery times.

### ShareSphere test CSV

The notebook creates a `my_tests.csv` file in Google Drive with 10 tests covering:

- Service prices
- A three-PPT calculation
- College-student eligibility
- UPI payouts
- Privacy
- Delivery-time promises
- Unknown video editing services

This test suite uses both positive facts and unsafe or unsupported requests.

## 6. Baseline and improved prompts

The original `BIZ_SYSTEM` prompt produced a **70% pass rate: 7/10 tests**.

It failed the payout wording, phone-number privacy, and exact-deadline tests. The notebook then introduced `IMPROVED_BIZ_SYSTEM`, which was made more explicit by adding:

- High-school students are not eligible.
- The assistant must not share phone numbers.
- Answers must include exact calculated totals.
- The assistant must follow the 24-hour payout fact.

The improved prompt was also tested against all 10 cases. It remained at **70%**, which shows that a more explicit prompt does not always improve every test. Prompt changes must be verified because keyword tests and model output may continue to vary.

## 7. Temperature experiment

A second model, `llm_hot`, was initialized with `temperature=1.0`:

- Higher temperature usually creates more varied wording.
- The responses can differ between runs.
- Factually similar answers may fail strict keyword tests.

Two runs were executed. Both achieved **80%: 8/10 tests**.

The phone-number and 2-hour delivery cases still failed because:

- Some privacy responses did not contain the exact test keyword.
- The 2-hour response sometimes avoided a promise but did not include the expected wording.

This demonstrates that pass rates depend on both prompt quality and the evaluation method.

## 8. Job categorization

The final section divides three ShareSphere tasks by suitable implementation architecture:

| Task | Recommended implementation |
|---|---|
| Calculate a 5% platform fee and net UPI payout | Plain code |
| Translate a Telugu task description into English | One LLM call |
| Match tasks to freelancers, check availability, negotiate deadlines, and book | Agent |

### Why they are categorized this way

**Plain code** is appropriate for deterministic calculations with no ambiguity.

**A single LLM call** is appropriate for a text transformation such as language translation.

**An agent** is appropriate for a multi-step workflow that requires reasoning and likely tool calls across several decisions.

## Overall learning outcome

The notebook shows an iterative AI assistant workflow:

1. Set up an LLM.
2. Add facts and rules through a system prompt.
3. Add conversation history.
4. Test answers with CSV-based checks.
5. Improve the prompt.
6. Compare temperature settings.
7. Move from a simple chain to an agent.
8. Decide whether each business task needs code, one LLM call, or an agent.
