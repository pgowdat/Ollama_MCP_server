# Build your own Local MCP Client with LlamaIndex

This project demonstrates how to build a **local MCP (Model Context Protocol) client** using LlamaIndex. The client connects to a local MCP server (which exposes tools like a SQLite database) and lets you interact with it using natural language and tool-calling agents—all running locally on your machine.


### Setup

To sync dependencies, run:

```sh
uv sync
```

---

## Usage

- Start the local MCP server (for example, the included SQLite demo server):

```sh
uv run server.py --server_type=sse
```

- Run the client (choose the appropriate client script, e.g. `client.py` for OpenAI or `ollama_client.py` for Ollama):

```sh
uv run client.py
```

# LlamaIndex MCP Agent: Chat with Your Database

This project lets you **converse with your SQLite database using natural language**, powered by a local AI model. You talk in plain English—your agent securely handles database operations for you via a modern, tool-based architecture.

## 🚦 What Is This Project?

You get an end-to-end, local system that lets you:
- **Ask questions about your data** or **add new data** conversationally.
- Ensure **safe, controlled access** to your database (never direct SQL exposure).
- Run everything—including the Large Language Model (LLM), the API server, and the agent—on your own machine.

## 💡 How Does It Work? (Brief Explanation)

The system has two main components:

### 1. **MCP Server (`server.py`)**
- Handles all database logic for a local SQLite file (`demo.db`).
- Exposes two safe "tools" over HTTP (not direct SQL!):
    - `add_data(query: str)`: Adds new data via SQL INSERT.
    - `read_data(query: str)`: Reads data via SQL SELECT.
- Receives and executes only these specific commands.

### 2. **LlamaIndex Agent Client (`client.py`)**
- Uses **Ollama** to run a local LLM (such as Llama 3.2).
- Starts up, discovers available tools from the MCP server.
- Listens for your input in plain English.
- Decides *which* tool to use and generates SQL as needed.
- Sends a safe request to the server, retrieves and presents the result conversationally.

## ✨ Key Features

- **Natural Language Chat:** No need to write SQL. Just say what you want!
- **Safe, Tool-Based Access:** Only two tools are exposed—no risky queries.
- **Full Local Control:** No remote servers needed. Your data and LLM run on your device.
- **Automatic Tool Discovery:** The client learns about available database actions at startup.
- **Separation of Concerns:** Server handles the database; client handles the AI and user chat.

## ⚙️ How It Works (Detail)

### System Flow

1. **MCP Server Setup:**
   - A SQLite database (`demo.db`) is initialized with a `people` table.
   - With `mcp-server`, the server exposes tool endpoints for adding and reading data.

2. **Client/Agent Operation:**
   - Powered by Ollama’s local LLM (e.g., llama3.2).
   - Discovers tools using a spec (`McpToolSpec`) that points to the server ([http://127.0.0.1:8000/sse](http://127.0.0.1:8000/sse)).
   - Handles queries like:
     - *You*: "Add a new person named Alice who is 25."
     - *Agent*: Generates `INSERT INTO people (name, age) VALUES ('Alice', 25)`, calls `add_data()`, and responds based on execution.

## 🚀 Getting Started

### Prerequisites

- **Python 3.9+**
- **Ollama** (installed and running—see [Ollama](https://ollama.com/))
- **Llama 3.2 model** pulled for Ollama:
    ```bash
    ollama pull llama3.2
    ```

### 1. Installation

```bash
# Clone the repo and move into it
# (Assuming you already did this step)

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate

# Install dependencies
pip install "mcp-server[fastmcp,sqlite]" llama-index llama-index-llms-ollama nest-asyncio uv
```

### 2. Start the MCP Server

```bash
uv run server.py --server_type=sse
```

- MCP server launches at `http://127.0.0.1:8000`.
- Exposes only `add_data` and `read_data` as tools.

### 3. Start the LlamaIndex Agent

Open a new terminal (do **not** stop the server!) and run:

```bash
python client.py
```

- The agent initializes and prompts you for input:  
  *Example:*
  ```
  Enter your message:
  ```

## 📦 Example Conversation

**You:**  
> Add a record for John Doe, age 30, who is an Engineer.

**Agent:**  
> Added John Doe (age 30, Engineer) to the database successfully.

**You:**  
> Who is in the database?

**Agent:**  
> Here are the current people: Alice (25), John Doe (30, Engineer), etc.

## 🧩 System Architecture

| Component      | Role                           | Implementation            |
|----------------|-------------------------------|---------------------------|
| `server.py`    | Secure DB Tool Server          | FastMCP, SQLite           |
| `client.py`    | AI Agent Chat Frontend         | LlamaIndex, Ollama LLM    |
| `demo.db`      | SQLite Database                | Table: people             |

## 📝 Notes

- **No direct DB access for AI:** The agent can only call safe "tools," never direct queries.
- **Customizable:** Add new tools or database fields as needed by modifying `server.py` and updating your schema.
- **All local:** Your data never leaves your device—full privacy.

## ❓ Why Use This Approach?

- **Security:** Prevents unwanted queries; restricts what AI can do with your database.
- **Natural Workflow:** End users need no technical knowledge—just ask as you would a person.
- **Modular:** Swap out the database, change the agent, or add more tools.

