# Agent Day 1
This notebook demonstrates creating an AI assistant with LangChain and Groq.
## Setup
```python
!pip install langchain_groq
```
## Packages
- `langchain_groq` provides the Groq chat integration.
- `groq` enables direct access to Groq models.
- `ChatGroq` initializes the LangChain chat model.
- Python 3.13 is used in the Colab environment.
## API Configuration
- The Groq API key is stored in Google Colab Secrets.
- The secret is retrieved under the name `Groq`.
- The key is assigned to `GROQ_API_KEY`.
## Bakery Assistant
- The assistant represents Sweet Crumbs Home Bakery.
- Orders are baked to order with at least 24-hour notice.
- Eggless cake versions are available.
- It never promises delivery times.
- Responses are polite and limited to under 60 words.
## Prompt Guard
- The notebook tests Groq's prompt guard model.
- It sends an instruction designed to bypass safety rules.
- The returned score indicates unsafe-content probability.
## Result
The notebook successfully combines LangChain, Groq, and prompt safety.
