# Week 3: Multi-Provider LLM Client & Usage Monitoring

Welcome to **Week 3**! 👋 In this lab, we build a production-ready, multi-provider LLM client in Python. It can communicate with either cloud-hosted models (like **Google Gemini**) or local open-source models (via **Ollama**), automatically retry on network failures, measure latency, calculate costs, and log all telemetry into a **MongoDB** database.

---

## 📖 Table of Contents
- [What You Will Learn](#-what-you-will-learn)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Step-by-Step Setup](#-step-by-step-setup)
- [How to Run](#-how-to-run)
- [Running Unit Tests](#-running-unit-tests)
- [Key Concepts Explained (Beginner Friendly)](#-key-concepts-explained-beginner-friendly)
- [Troubleshooting & FAQs](#-troubleshooting--faqs)

---

## 🎯 What You Will Learn
1. **Provider Abstraction**: How to write clean, reusable code that works with different AI providers without changing your core application logic.
2. **Hosted vs. Local LLMs**: The difference between calling a hosted API (Google Gemini) vs. running a model locally on your own computer (Ollama).
3. **Resilience & Retries**: Handling API hiccups gracefully with exponential backoff (retrying after 1s, 2s, 4s).
4. **Usage Telemetry & Logging**: Tracking token consumption (input vs. output tokens), latency in milliseconds, and estimated costs, and persisting records in MongoDB.

---

## 📂 Project Structure

```text
Week3/
├── llm_client.py       # Core logic: Provider classes (Gemini & Ollama), retry logic & cost calculation
├── usage_logger.py     # MongoDB logger: Connects to MongoDB and saves usage history
├── main.py             # User interface: Interactive terminal app to ask questions
├── test_client.py      # Unit tests: Tests client logic using fake providers/loggers
├── requirements.txt    # Python dependencies
├── .env                # Environment variables (API keys, DB connection - do NOT commit)
└── .gitignore          # Ignores sensitive and cached files
```

---

## 🛠️ Prerequisites
Before running the code, make sure you have:
1. **Python 3.10+** installed on your system.
2. **Gemini API Key**: Get a free API key from [Google AI Studio](https://aistudio.google.com/).
3. **MongoDB Database**: A free MongoDB Atlas cluster connection string or a local MongoDB server (`mongodb://localhost:27017`).
4. *(Optional)* **Ollama**: If you want to run models locally, download Ollama from [ollama.com](https://ollama.com/) and run `ollama pull llama3.2`.

---

## 🚀 Step-by-Step Setup

### Step 1: Open Terminal in Week 3 Folder
Navigate into the `Week3` directory:
```bash
cd "2520030381_GENAI/Week3"
```

### Step 2: Create and Activate a Virtual Environment
It is best practice to keep dependencies isolated:
- **Windows (Command Prompt / PowerShell):**
  ```powershell
  python -m venv .venv
  .venv\Scripts\Activate.ps1
  ```
- **macOS / Linux:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  ```

### Step 3: Install Required Packages
Install the required packages using `pip`:
```bash
pip install -r requirements.txt
```

### Step 4: Configure Your `.env` File
Create a `.env` file in the `Week3` directory (or edit the existing one) and fill in your credentials:

```ini
# Google Gemini API Key
GEMINI_API_KEY=your_gemini_api_key_here

# MongoDB Connection String (Atlas or Local)
MONGODB_URI=mongodb://localhost:27017
# Example Atlas URI: mongodb+srv://<username>:<password>@cluster0.mongodb.net/?retryWrites=true&w=majority

# Pricing per 1 Million Tokens (USD)
GEMINI_INPUT_PRICE_PER_MILLION=0.15
GEMINI_OUTPUT_PRICE_PER_MILLION=0.60
```

---

## ▶️ How to Run

Run the interactive program:
```bash
python main.py
```

### What happens when you run it:
1. **Choose Provider**: The program asks whether you want to use `gemini` or `ollama`.
2. **Enter Prompt**: Type your question (e.g., *"What is machine learning?"*).
3. **Execution & Metrics**:
   - The program calls the chosen LLM.
   - Measures response latency in milliseconds.
   - Calculates prompt & response tokens.
   - Computes estimated cost (hosted API only; local Ollama has 0 API cost).
   - Saves a record to MongoDB in database `llm_monitoring`, collection `api_usage`.
4. **Response**: Prints the AI's response in your terminal!

---

## 🧪 Running Unit Tests

Unit tests ensure your code works as expected without calling real APIs or spending credits. Run the tests using:

```bash
python -m unittest test_client.py
```
*or if you have `pytest` installed:*
```bash
pytest test_client.py
```

You should see an output like:
```text
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
```

---

## 💡 Key Concepts Explained (Beginner Friendly)

### 1. What is an LLM Provider Abstraction?
Different companies provide different APIs. Instead of rewriting your entire code every time you switch from Gemini to Ollama or OpenAI, we use an **Abstract Base Class (`LLMProvider`)**. Both `GeminiProvider` and `OllamaProvider` implement a common `.generate(prompt)` method that returns standard `LLMResult` objects.

### 2. What are Tokens?
Tokens are chunks of characters that LLMs read and produce.
- **Input Tokens (Prompt)**: The words/tokens you send to the model.
- **Output Tokens (Candidates / Completion)**: The words/tokens generated by the model.
- Providers charge per million tokens, so measuring them helps track and optimize costs.

### 3. Exponential Backoff Retries
If the API server is overloaded (e.g., HTTP 429 or 503 error), the client waits before retrying:
- 1st retry: wait $2^0 = 1$ second
- 2nd retry: wait $2^1 = 2$ seconds
- 3rd retry: wait $2^2 = 4$ seconds
This prevents spamming a struggling server.

### 4. Database Logging
Every request is recorded in MongoDB with:
- Timestamp (UTC)
- Model & provider used
- Input tokens, output tokens, total tokens
- Latency (time taken in ms)
- Success or failure status and error details

---

## ❓ Troubleshooting & FAQs

- **Error: `Gemini API key not found in .env`**
  - Make sure you created a `.env` file in the `Week3` directory and named the key `GEMINI_API_KEY`.
- **Error: `MONGODB_URI not found in .env` or MongoDB timeout**
  - Ensure your MongoDB connection string is correct and your database is running. If using MongoDB Atlas, check that your IP address is whitelisted under Network Access in Atlas.
- **Error when selecting Ollama**:
  - Make sure the Ollama application is running (`ollama serve`) and you have pulled the model (`ollama pull llama3.2`).
