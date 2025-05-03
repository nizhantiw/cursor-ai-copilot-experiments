
# AI Code Autocomplete

A web-based AI autocomplete tool that provides code suggestions as you type, similar to GitHub Copilot, using locally-hosted LLMs through Ollama.

## Features

*   Integrates with Monaco Editor (Visual Studio Code's editor) for a familiar coding experience.
*   Sends relevant code context (including semantic examples) to locally hosted Ollama LLMs.
*   Supports both code completion and code generation from comment instructions.
*   Uses `nomic-embed-text` for semantic understanding of code context.
*   Shows predictions as interactive ghost text.
*   Accept completions with Tab or trigger manually.
*   Supports multiple programming languages (JS, TS, Python, Java, C#, C++).
*   Works with various Ollama models (Llama 3.2, DeepSeek Coder, etc.).
*   Includes basic repetition and hallucination detection.
*   Adaptive debounce timing for responsiveness.

## Prerequisites

*   Node.js (v18+) and npm installed.
*   [Ollama](https://ollama.ai/) installed and running.
*   Required Ollama models pulled:
    *   A completion model (e.g., `ollama pull llama3.2`)
    *   The embedding model (`ollama pull nomic-embed-text`)
*   Optional: Other completion models (`ollama pull deepseek-coder`, etc.)

## Setup

1.  **Clone the repository:**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-name>
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    ```
3.  **Generate Context Embeddings:**
    *   Ensure Ollama is running with the `nomic-embed-text` model available.
    *   Run the preprocessing script:
        ```bash
        node generate_embeddings.js
        ```
    *   This creates `server/embeddings/js_context_embeddings.json`.
4.  **Start the Server:**
    *   Ensure Ollama is running.
    *   Start the backend server:
        ```bash
        npm start
        ```
    *   The server will load the embeddings and connect to Ollama.
5.  **Access the Application:** Open your web browser to `http://localhost:5000` (or the configured port).

## Architecture & Tech Stack

*   **Frontend (`public/`)**: Pure HTML, CSS, and JavaScript.
    *   **Editor**: Monaco Editor loaded via CDN for the code editing interface.
    *   **Logic (`app.js`)**: Handles editor events, context extraction (full code, snippet), debouncing, partial keyword detection, API calls, and ghost text display.
    *   **Context (`javascript_context.js`)**: Provides static code examples used for generating semantic embeddings.
*   **Backend (`server/server.js`)**: Node.js with Express.js.
    *   **API**: Defines endpoints (`/api/completion`, `/api/health`) to handle requests from the frontend.
    *   **Ollama Interaction**: Uses `node-fetch` to communicate with the local Ollama service for both `/api/generate` (completion/generation) and `/api/embeddings` (semantic search).
    *   **Embedding Handling**: Loads pre-computed embeddings from JSON on startup, calculates cosine similarity for semantic search.
    *   **Prompt Logic**: Dynamically constructs prompts based on request type (completion/generation) and semantic search results.
    *   **Static Serving**: Serves the frontend files.
*   **AI Backend (Local)**: Ollama service.
    *   **Completion/Generation Model**: E.g., `llama3.2` - Responsible for generating code suggestions or full code blocks.
    *   **Embedding Model**: `nomic-embed-text` - Used to create vector representations of code for semantic understanding.
*   **Data (`server/embeddings/`)**: Stores pre-computed embeddings in `js_context_embeddings.json`.

## Functionality Overview (Technical)

1.  **Input & Triggering**: As the user types in Monaco, `app.js` listens for changes. A debounce mechanism (`handleEditorContentChange`) waits for a pause in typing. This delay is shortened if a potential keyword prefix (like `as`, `fu`) is detected (`_detectPartialKeyword`), triggering the process faster.
2.  **Context Extraction**: Before calling the API, `_extractContext` gathers:
    *   `code`: The full content of the editor file (potentially truncated and including static JS context).
    *   `snippet`: A small window of code around the cursor (e.g., 5 lines before, 3 after).
3.  **API Call**: The frontend sends `code`, `snippet`, `language`, and the selected `model` to the backend `/api/completion` endpoint.
4.  **Backend Processing**: The server determines the request type:
    *   **Instruction Comment**: If the snippet starts like `// write me...`, it builds a *generation* prompt using the comment's instruction and the `code` context.
    *   **Standard Completion**: Otherwise, it generates an embedding for the `snippet`, performs a cosine similarity search against the loaded context embeddings, finds the most relevant `chunk`, and builds a *completion* prompt including the relevant `chunk`, `code`, and `snippet`.
5.  **Ollama Inference**: The backend sends the constructed prompt to the appropriate Ollama model (`llama3.2` for completion/generation).
6.  **Response & Cleaning**: Ollama returns the raw text. The backend performs basic cleaning (removes markdown, trims snippet prefix for completions).
7.  **Ghost Text Display**: The cleaned completion is sent back to the frontend, which displays it as ghost text using Monaco's decoration API or fallback methods.
8.  **Acceptance**: Pressing Tab accepts the ghost text, inserting it into the editor.

## Usage

1.  Ensure Ollama is running with the required models (`llama3.2`, `nomic-embed-text`).
2.  Run `npm start` to launch the server.
3.  Open `http://localhost:5000` in your browser.
4.  Start typing JavaScript code.
5.  Observe ghost text suggestions appearing.
6.  Press `Tab` to accept a suggestion.
7.  Try typing instruction comments like `// create a function to add two numbers` and wait/trigger completion. 
