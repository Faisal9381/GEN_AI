# Week 4: Exploring Tokenizers & Subword Algorithms

Welcome to **Week 4**! 👋 In this lab, we dive into how Large Language Models actually "read" and process text: **Tokenization**.

Before an LLM can understand sentences, words are split into numerical tokens. Here, we compare **three major tokenization algorithms** across **English and Indic languages (Telugu and Hindi)** to see how each handles different scripts and vocabularies.

---

## 📖 Table of Contents
- [What You Will Learn](#-what-you-will-learn)
- [Project Structure](#-project-structure)
- [The Three Tokenizers Compared](#-the-three-tokenizers-compared)
- [Step-by-Step Setup](#-step-by-step-setup)
- [How to Run](#-how-to-run)
- [Sample Output & What to Notice](#-sample-output--what-to-notice)
- [Key Concepts Explained (Beginner Friendly)](#-key-concepts-explained-beginner-friendly)
- [The "Token Tax" on Multilingual Texts](#-the-token-tax-on-multilingual-texts)

---

## 🎯 What You Will Learn
1. **How LLMs see text**: Text $\rightarrow$ Tokens $\rightarrow$ Token IDs (integers).
2. **Comparing Tokenization Algorithms**:
   - **BPE (Byte Pair Encoding)**: Used by GPT models.
   - **WordPiece**: Used by BERT models.
   - **SentencePiece (Unigram)**: Used by XLM-RoBERTa, LLaMA, and Gemini models.
3. **The Multilingual "Token Tax"**: Why non-English languages often require significantly more tokens for the exact same sentence, impacting latency and API cost.

---

## 📂 Project Structure

```text
Week4/
├── tokenizer_comparision.py  # Python script comparing tokenizers on English, Telugu, and Hindi
├── requirements.txt         # Required libraries: transformers and sentencepiece
└── README.md                # This guide
```

---

## 🔍 The Three Tokenizers Compared

| Tokenizer Algorithm | Model Used in Script | Primary Usage | How it Works |
| :--- | :--- | :--- | :--- |
| **BPE (Byte Pair Encoding)** | `gpt2` | GPT series (OpenAI) | Starts with individual bytes/characters and iteratively merges the most frequent pairs. |
| **WordPiece** | `bert-base-multilingual-cased` | BERT series (Google) | Maximizes the likelihood of the training data when choosing pieces to merge. Uses `##` for subwords. |
| **SentencePiece (Unigram)** | `xlm-roberta-base` | XLM-R, LLaMA, Gemini | Treats whitespace like a character (` `) and chooses the optimal subword segmentation using probability. |

---

## 🚀 Step-by-Step Setup

### Step 1: Open Terminal in Week 4 Folder
Navigate into the `Week4` directory:
```bash
cd "2520030381_GENAI/Week4"
```

### Step 2: Create and Activate a Virtual Environment
- **Windows (PowerShell):**
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
Install the `transformers` and `sentencepiece` libraries:
```bash
pip install -r requirements.txt
```
*(Note: Downloading tokenizers from Hugging Face Hub during first run requires an active internet connection).*

---

## ▶️ How to Run

Execute the comparison script:
```bash
python tokenizer_comparision.py
```

---

## 📊 Sample Output & What to Notice

When the script runs, it processes three identical sentences in English, Telugu, and Hindi:

### 1. English Input:
> *"Artificial intelligence is changing the world."*
- **GPT-2 (BPE)**: ~7 tokens
- **mBERT (WordPiece)**: ~8 tokens
- **XLM-RoBERTa (SentencePiece)**: ~8 tokens
*Notice: English words are common in vocabularies, so they map cleanly to 1 token per word (or common subwords).*

### 2. Telugu Input:
> *"కృత్రిమ మేధస్సు ప్రపంచాన్ని మారుస్తోంది."*
- **GPT-2 (BPE)**: ~40–60 tokens! (GPT-2 was not trained on Telugu, so it falls back to raw bytes `\xe0\xb0...`).
- **mBERT**: ~25 tokens.
- **XLM-RoBERTa**: ~12–15 tokens (Handles multilingual scripts much more efficiently).

### 3. Hindi Input:
> *"कृत्रिम बुद्धिमत्ता दुनिया को बदल रही है।"*
- Similar to Telugu, multilingual tokenizers (WordPiece / SentencePiece) tokenize Hindi words into meaningful subword roots, whereas older English-centric tokenizers fragment them heavily.

---

## 💡 Key Concepts Explained (Beginner Friendly)

### 1. What is a Token?
Computers cannot process raw text directly. Tokenization breaks a string of text into smaller units:
```text
"Artificial" -> Token ID: 32364
" intelligence" -> Token ID: 4430
```
These IDs are then used to look up vector embeddings inside the neural network.

### 2. Characters vs. Words vs. Subwords
- **Word Tokenization**: Fails on typos, new words, or compound words (requires an infinite dictionary).
- **Character Tokenization**: Too granular; loses semantic meaning and makes sequences very long.
- **Subword Tokenization (The Standard)**: Common words stay whole (`"changing"`), while rare or complex words get split into known chunks (`"micro"` + `"processor"`).

### 3. The "Token Tax" Explained
If a prompt in English takes **100 tokens** but the exact same meaning in Telugu or Hindi takes **350 tokens**:
- 💸 **Cost**: You pay $3.5\times$ more for API calls in that language.
- ⏱️ **Speed**: The model takes longer to generate the output because it outputs one token at a time.
- 📦 **Context Window**: Your prompt uses up more of the model's memory limit.

This is why modern foundation models (like Gemini and LLaMA-3) train expanded multilingual tokenizers!
