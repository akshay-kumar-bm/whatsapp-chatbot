# WhatsApp Business Chatbot

An AI customer-support chatbot for small businesses: it answers product and FAQ questions on WhatsApp (via Twilio), with a Flask admin portal and an analytics dashboard.

## What it does
A business owner logs in to a web portal, enters business info, products and FAQs, and the bot answers customers' WhatsApp messages using a Groq-hosted Llama model grounded on that data through a Chroma vector store. Conversations are logged and summarised in a metrics dashboard.

## Features (as implemented)
- Twilio webhook `/twilio/receiveMessage` that replies to incoming WhatsApp messages
- `ProductChatbot` (LangChain + `ChatGroq`, model `llama-3.1-70b-versatile`, conversation buffer memory) with a prompt covering knowledge-base answers and product recommendations
- ChromaDB vector store with HuggingFace embeddings (`vector_store.py`) for products and FAQs
- Admin portal: register/login (Flask-Login, bcrypt-hashed passwords), CRUD routes for business info, products and FAQs, backed by JSON files
- Chat-history logging and `/chat_history/<user_id>`
- `/metrics_dashboard` + `/get_metrics` with keyword-based sentiment counts and conversation stats
- Optional Streamlit front end (`streamlit_frontend.py`)

## Tech stack
Flask, Flask-Login, Flask-Bcrypt, Twilio, LangChain, Groq, ChromaDB, HuggingFace embeddings (llama-index), Streamlit, vanilla HTML/CSS/JS.

## Flow
```
WhatsApp -> Twilio -> /twilio/receiveMessage (Flask) -> ProductChatbot -> Chroma retrieval + Groq LLM
        <- Twilio reply <-                              chat_history.json -> /metrics_dashboard
```

## Setup
```bash
pip install -r requirments.txt
```
Environment variables (names read in code): `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER`, `GROQ_API_KEY` (put them in a git-ignored `.env`).
```bash
python app.py        # Flask on port 5000, debug mode
```
Expose the server (e.g. with ngrok) and set the Twilio WhatsApp sandbox webhook to `/twilio/receiveMessage`.

## Structure
```
app.py            routes, auth, metrics, Twilio webhook
chatbot.py        LLM chatbot        vector_store.py   Chroma store
tools.py          chat-history helpers   streamlit_frontend.py
templates/ static/                  business_info.json FAQ.json inventory.json users.json
```

## Limitations
- Data is stored in JSON files, not a database; no `init_db.py` exists despite older README notes.
- Sentiment analysis is keyword-based; "multi-language", "satisfaction" and "conversion" claims in the old README are not clearly backed by code.
- `streamlit_frontend.py` imports names not defined in `chatbot.py` and appears stale.
- Code references "FlowerShopVectorStore" from an earlier flower-shop version; requirements file is misspelled and unpinned.
- Repo contains committed runtime data and large binaries (see verdict).
