# LangChain AI Assistant and Tool-Calling Notebook

## Overview

This notebook demonstrates how to build practical AI assistants with LangChain and Groq. It contains two applications:

1. **Sweet Crumbs Home Bakery** - answers menu, price, timing, and order questions.
2. **ShareSphere** - handles student service bookings, prices, mentor availability, and Google Sheets storage.

Both applications combine a large language model with Python tools and Pydantic validation. The LLM understands natural-language requests, while deterministic Python functions perform calculations and database operations.

## Libraries and Setup

The notebook installs and imports:

- `langchain` for the agent framework
- `langchain-groq` for Groq-hosted language models
- `gspread` for Google Sheets access
- `pydantic` for structured data validation
- `google.colab` helpers for authentication and secrets

The Groq API key is read from a Colab secret named `Groq` and used to create a `ChatGroq` model:

```python
llm = ChatGroq(
    model="openai/gpt-oss-20b",
    temperature=0.2,
    max_retries=3
)
```

A low temperature makes responses more consistent. Google Colab authenticates the user and creates a `gspread` client that can open or create spreadsheets.

## Sweet Crumbs Bakery Assistant

### Menu and Hours

The bakery menu is stored as a dictionary:

| Item | Price |
|---|---:|
| Plum cake | Rs 500 |
| Chocolate cake | Rs 650 |
| Cupcake | Rs 80 |

Opening hours are also stored by day. The bakery is open from 9 AM to 8 PM Monday through Saturday and from 10 AM to 6 PM on Sunday.

### Order Schema

The `Order` Pydantic model validates records before they are saved:

- `name`: customer's first name
- `item`: one of the three menu items
- `qty`: a positive integer
- `date`: pickup date
- `eggless`: whether the order should be eggless

Pydantic prevents invalid values, such as a quantity of zero or an item that is not on the menu.

### Bakery Tools

Three LangChain tools are provided:

1. `get_price(item, qty)` calculates the item price times quantity.
2. `is_open(day)` returns the opening hours for a valid weekday.
3. `log_order(...)` validates and saves a confirmed order.

The price and opening-hours tools are read-only. The order tool writes a row containing the customer's name, item, quantity, date, eggless preference, and total to the `Sweet Crumbs Orders` sheet.

The created or opened sheet is available at:

https://docs.google.com/spreadsheets/d/1Nji6clh-WP5SzsNk09_gTtETlZMCjBkXIUwih7ykKug

### Bakery Agent Behavior

The `SYSTEM` instructions tell the agent to:

- act as a friendly bakery assistant
- require 24 hours' notice
- use tools instead of guessing prices or hours
- collect name, item, quantity, and pickup date before logging
- read the saved order and total back to the customer
- keep replies under 60 words

Conversation history is passed back to the agent on every call, so it can remember earlier parts of the same chat.

The edge-case tests check:

- an unavailable menu item
- a missing pickup date
- an invalid quantity of zero

In these cases, the agent should explain the problem or request missing information instead of creating a bad order.

## ShareSphere Booking Assistant

### Service Catalogue

ShareSphere provides four services:

| Service | Rate per task |
|---|---:|
| Web development | Rs 800 |
| UI/UX | Rs 500 |
| PPT | Rs 250 |
| Data analytics | Rs 700 |

The ShareSphere sheet is named `ShareSphere Bookings`. If it does not exist, the notebook creates it and adds these headers:

```text
student_name, service, tasks, deadline, urgent, total
```

### Booking Schema

The `ServiceBooking` Pydantic model represents a validated booking:

- `student_name`: student's first name
- `service`: a supported ShareSphere service
- `tasks`: positive number of tasks or deliverables
- `deadline`: requested delivery date
- `urgent`: whether express delivery was requested

### Structured Output

`llm.with_structured_output(ServiceBooking)` forces the model to convert messy chat text into the Pydantic schema. The notebook sends five informal student messages and prints one validated record for each message.

This demonstrates structured extraction. Instead of receiving only free-form text, the application receives typed fields that can be used safely for validation and spreadsheet storage.

### ShareSphere Tools

The agent has three tools:

- `get_service_price(service, tasks)` calculates the service total.
- `check_mentor_hours(day)` returns campus mentor support timings.
- `log_booking(...)` validates and saves a booking.

The booking tool is not called until the student has supplied the name, service, task count, and deadline.

## Tool-Name and Docstring Quality

The notebook demonstrates that tool metadata affects LLM tool selection.

A deliberately poor tool is named `tool1` and has the vague description `does stuff`. The agent is then asked for a price. It does not have reliable semantic information for deciding that `tool1` should perform the price lookup.

The agent is restored with the clearly named `get_service_price` tool and a detailed docstring that explains:

- what the tool does
- which services are accepted
- what the `tasks` argument means

Clear names and descriptions act as a contract between the Python tool and the language model.

## Multi-Tool Execution Trace

The notebook also sends a combined question:

> Are you open on Sunday and what is the cost of 2 web dev tasks?

The agent can make separate tool calls:

1. call `check_mentor_hours("sunday")`
2. call `get_service_price("web dev", 2)`
3. combine both tool results into one final response

`pretty_print()` displays the sequence of messages, including user input, AI tool calls, tool results, and the final answer.

The trace summary counts:

- model/AI messages, representing LLM invocations
- tool messages, representing executed tools

This makes it possible to see exactly how the agent reasons and which operations it performs.

## Agent and Tool Flow

The overall flow is:

```text
Customer message
      |
      v
LangChain agent + system instructions
      |
      v
LLM decides whether a tool is needed
      |
      +--> get_price / get_service_price
      |
      +--> is_open / check_mentor_hours
      |
      +--> log_order / log_booking
                    |
                    v
             Pydantic validation
                    |
                    v
             Google Sheets row
```

## Key Lessons

- LangChain tools let an LLM perform calculations and actions through Python functions.
- Pydantic models validate structured information before it is stored.
- A clear system prompt defines the assistant's role and safety rules.
- Tool names and docstrings are important because the model reads them when deciding what to call.
- Google Sheets can act as a simple persistent database for orders and bookings.
- Conversation history enables follow-up questions.
- Execution traces are useful for debugging tool selection and multi-step agent behavior.
