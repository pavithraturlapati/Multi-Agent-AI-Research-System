# Multi Agent AI Research System

An agentic workflow that automates web research like searching, scraping, synthesizing, and critiquing reports on any topic. Built with **LangChain (LCEL)**, **Mistral AI**, and **Streamlit**.

## 🛠️ Architecture

ResearchMind uses a **shared state (blackboard) pattern** to coordinate a team of agents and chains instead of passing raw prompt strings between calls.

```mermaid
graph TD
    User([User Input: Topic]) --> State[Shared State Memory]
    State -->|Context| SearchAgent[Search Agent] --> Tavily[Tavily Search API]
    Tavily --> SearchAgent --> State
    State -->|Target URLs| ReaderAgent[Reader Agent] --> BS4[BeautifulSoup Scraper]
    BS4 --> ReaderAgent --> State
    State --> WriterChain[Writer Chain] --> State
    State --> CriticChain[Critic Chain] --> State
    State --> StreamlitUI[Streamlit UI]
```
## ⚙️ How It Works

The system processes your research topic in five coordinated stages:

**1. Shared State Memory** — A centralized `state = {}` dictionary (blackboard pattern) coordinates all data across agents and chains, tracking `search_results`, `scraped_content`, `report`, and `feedback`.

**2. Search Agent** — Built with LangChain's `create_agent`, the Mistral-powered agent reasons over and invokes the custom `web_search` tool, querying the **Tavily Search API** with `max_results=5` for high-precision results within a controlled context budget.

**3. Reader Agent** — Ingests target URLs from shared state and deep-scrapes each page using **BeautifulSoup4**. Employs spoofed browser headers to bypass bot detection, an 8-second timeout for resiliency, aggressive boilerplate stripping (`<script>`, `<nav>`, `<footer>`), and a hard 3,000-word cap per page to prevent context-window overflow.

**4. Writer Chain** — An LCEL pipeline that synthesizes all scraped findings into a structured Markdown report covering an Executive Summary, Key Findings (minimum 3 themes), Conclusion, and a Bibliography referencing original sources.

**5. Critic Chain** — Independently audits the generated report and returns structured JSON feedback containing a score out of 10, identified strengths, areas of improvement, and a single-sentence editorial verdict.


## 💻 Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | LangChain (`create_agent`, LCEL) |
| LLM | Mistral AI |
| Search | Tavily Search API |
| Scraping | BeautifulSoup4 + Requests |
| UI | Streamlit |
| Logging | Rich |
| Config | python-dotenv |

## 📂 Project Structure

```
multi_agent_system/
├── .env                  # API credentials (not committed)
├── requirements.txt
├── tools.py              # Tavily search + BS4 scraping tools
├── agents.py             # Agent + LCEL Writer/Critic chains
├── pipeline.py           # Orchestration & shared state
└── app.py                # Streamlit dashboard
```
