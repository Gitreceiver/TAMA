# My Python Task Manager (TAMA)

## Overview

TAMA (Task Assistant & Manager) is an AI-powered command-line task manager designed to help break down complex projects, manage tasks and subtasks, and leverage AI for task generation and expansion. It uses DeepSeek AI models via the OpenAI client library.

## Features

*   **Task Management:** Create, list, view, and update the status of tasks and subtasks.
*   **Hierarchical Tasks:** Supports nested subtasks.
*   **Dependency Tracking:** Define dependencies between tasks and subtasks.
*   **Prioritization:** Assign high, medium, or low priority to tasks.
*   **AI Task Generation:** Parse Product Requirements Documents (PRDs) using AI to automatically generate a task list (`parse-prd` command).
*   **AI Task Expansion:** Break down complex tasks into smaller subtasks using AI (`expand` command).
*   **Next Task Suggestion:** Find the next available task based on status and dependencies (`next` command).
*   **Dependency Check:** Detect circular dependencies (`check-deps` command).
*   **Complexity Estimation:** Get a simple complexity estimate for tasks (`show` command).
*   **File Generation:** Generate placeholder code/markdown files from tasks (`generate-file` command).
*   **Rich CLI:** Uses `rich` for formatted tables and detailed views.

## Installation

This project uses [uv](https://astral.sh/blog/uv) for dependency management.

1.  **Clone the repository:**
    ```bash
    git clone <your-repo-url>
    cd my-python-task-manager # Or your repo name
    ```

2.  **Install dependencies:**
    ```bash
    # Install dependencies from uv.lock or pyproject.toml
    uv pip sync pyproject.toml
    ```

3.  **Activate virtual environment:**
    *   If `uv` created a `.venv` directory:
        *   Windows: `.venv\Scripts\activate`
        *   macOS/Linux: `source .venv/bin/activate`
    *   If using another virtual environment manager (like `conda` or standard `venv`), activate it accordingly.

## Configuration

Configuration is managed via environment variables using a `.env` file.

1.  Copy the example file:
    ```bash
    cp .env.example .env
    ```

2.  **Edit `.env`:**
    *   Add your `DEEPSEEK_API_KEY`. You can obtain one from [DeepSeek AI](https://platform.deepseek.com/).
    *   (Optional) Adjust `DEEPSEEK_BASE_URL` if needed.
    *   (Optional) Change model names (`DEEPSEEK_GENERAL_MODEL`, `DEEPSEEK_REASONING_MODEL`).
    *   (Optional) Modify other settings like `LOG_LEVEL`, `DEFAULT_PRIORITY`, `TASKS_JSON_PATH`, etc.

    ```ini
    # .env example content
    DEEPSEEK_API_KEY="YOUR_DEEPSEEK_API_KEY_HERE"
    DEEPSEEK_BASE_URL="https://api.deepseek.com/v1" # Default

    DEEPSEEK_GENERAL_MODEL="deepseek-chat"
    DEEPSEEK_REASONING_MODEL="deepseek-coder"

    LOG_LEVEL="INFO"
    DEFAULT_PRIORITY="medium"
    PROJECT_NAME="My Task Manager Project"
    DEBUG=False
    TASKS_JSON_PATH="tasks.json" # Default location for tasks
    TASKS_DIR_PATH="tasks" # Not currently used, but defined
    AI_TEMPERATURE=0.7
    AI_MAX_TOKENS=8192
    ```

## Usage

All commands should be run from the project root directory with the virtual environment activated.

**1. Activate the virtual environment:**

   * If `uv` created a `.venv` directory:
        *   Windows: `.venv\Scripts\activate`
        *   macOS/Linux: `source .venv/bin/activate`

**2. Run commands:**

   The main entry point is `src/cli/main.py`. You can run commands using `python -m src/cli/main <command> [OPTIONS]`.

**Core Commands:**

*   **List tasks:**
    ```bash
    python -m src/cli/main list
    python -m src/cli/main list --status pending
    python -m src/cli/main list --priority high
    ```

*   **Show task details:**
    ```bash
    python -m src/cli/main show 1      # Show task 1
    python -m src/cli/main show 3.1    # Show subtask 1 of task 3
    ```
    *(This will also show the estimated complexity)*

*   **Set task status:**
    ```bash
    python -m src/cli/main set-status --id 2 --status done
    python -m src/cli/main set-status --id 1 --id 3.1 --status in-progress
    ```

*   **Find next task:**
    ```bash
    python -m src/cli/main next
    ```

*   **Check dependencies:**
    ```bash
    python -m src/cli/main check-deps
    ```

*   **Generate placeholder file:**
    ```bash
    python -m src/cli/main generate-file 1
    python -m src/cli/main generate-file 2 --output-dir ./code_files
    ```

**AI Commands:**

*   **Parse PRD:**
    ```bash
    # Create a my_project.prd file first
    python -m src/cli/main parse-prd my_project.prd
    ```

*   **Expand Task:**
    ```bash
    python -m src/cli/main expand 3 # Expand task 3 into subtasks
    ```

## Architecture Overview

*   **`src/config`:** Handles loading settings from `.env` using Pydantic.
*   **`src/task_manager`:** Contains core logic:
    *   `data_models.py`: Pydantic models for tasks, subtasks, etc.
    *   `storage.py`: Functions for loading/saving tasks from/to JSON.
    *   `core.py`: Core task manipulation functions (get, set status, find next).
    *   `dependencies.py`: Circular dependency detection.
    *   `complexity.py`: Basic complexity estimation.
    *   `file_generator.py`: Placeholder file generation.
    *   `parsing.py`: Logic for parsing PRD using AI.
    *   `expansion.py`: Logic for expanding tasks using AI.
*   **`src/ai`:** Handles AI interactions:
    *   `client.py`: Core function to call the DeepSeek API (via OpenAI client) with retries.
    *   `prompts.py`: Defines prompts used for AI generation.
*   **`src/cli`:** Command-line interface:
    *   `main.py`: Defines Typer commands and orchestrates calls to other modules.
    *   `ui.py`: Uses `rich` to display formatted output (tables, details).
*   **`tests`:** Contains unit and integration tests using `pytest`.
*   **`tasks.json`:** Default file for storing task data.

## Contributing

Contributions are welcome! Please follow standard coding practices and ensure tests pass.

*(Add more details here if needed: branching strategy, PR process, etc.)*

## Contributing

Contributions are welcome! Please follow standard coding practices and ensure tests pass.

To run tests:

```bash
uv run pytest tests/unit
uv run pytest tests/integration
```

*(Add more details here if needed: branching strategy, PR process, etc.)*

## License

*(Specify your license here, e.g., MIT License)*