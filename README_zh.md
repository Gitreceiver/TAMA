# 我的 Python 任务管理器 (TAMA)

## 概述

TAMA (Task Assistant & Manager) 是一个 AI 驱动的命令行任务管理器，旨在帮助分解复杂项目、管理任务和子任务，并利用 AI 进行任务生成和扩展。它使用 DeepSeek AI 模型，并通过 OpenAI 客户端库进行交互。

## 功能

*   **任务管理:** 创建、列出、查看和更新任务和子任务的状态。
*   **层级任务:** 支持嵌套的子任务。
*   **依赖跟踪:** 定义任务和子任务之间的依赖关系。
*   **优先级排序:** 为任务分配高、中或低优先级。
*   **AI 任务生成:** 使用 AI 解析产品需求文档 (PRD)，以自动生成任务列表 (`parse-prd` 命令)。
*   **AI 任务扩展:** 使用 AI 将复杂任务分解为更小的子任务 (`expand` 命令)。
*   **下一个任务建议:** 根据状态和依赖关系查找下一个可用的任务 (`next` 命令)。
*   **依赖检查:** 检测循环依赖 (`check-deps` 命令)。
*   **复杂度估计:** 获取任务的简单复杂度估计 (`show` 命令)。
*   **文件生成:** 从任务生成占位符代码/markdown 文件 (`generate-file` 命令)。
*   **丰富的 CLI:** 使用 `rich` 库进行格式化的表格和详细视图输出。

## 安装

本项目使用 [uv](https://astral.sh/blog/uv) 进行依赖管理。

1.  **克隆仓库:**
    ```bash
    git clone <你的仓库 URL>
    cd my-python-task-manager # 或你的仓库名称
    ```

2.  **安装依赖:**
    ```bash
    # 从 uv.lock 或 pyproject.toml 安装依赖
    uv pip sync pyproject.toml
    ```

3.  **激活虚拟环境:**
    *   如果 `uv` 创建了一个 `.venv` 目录:
        *   Windows: `.venv\Scripts\activate`
        *   macOS/Linux: `source .venv/bin/activate`
    *   如果使用其他虚拟环境管理器 (例如 `conda` 或标准的 `venv`), 请相应地激活它。

## 配置

配置通过环境变量使用 `.env` 文件进行管理。

1.  复制示例文件:
    ```bash
    cp .env.example .env
    ```

2.  **编辑 `.env`:**
    *   添加你的 `DEEPSEEK_API_KEY`。你可以从 [DeepSeek AI](https://platform.deepseek.com/) 获取一个。
    *   (可选) 如果需要，调整 `DEEPSEEK_BASE_URL`。
    *   (可选) 更改模型名称 (`DEEPSEEK_GENERAL_MODEL`, `DEEPSEEK_REASONING_MODEL`)。
    *   (可选) 修改其他设置，例如 `LOG_LEVEL`, `DEFAULT_PRIORITY`, `TASKS_JSON_PATH` 等。

    ```ini
    # .env 示例内容
    DEEPSEEK_API_KEY="YOUR_DEEPSEEK_API_KEY_HERE"
    DEEPSEEK_BASE_URL="https://api.deepseek.com/v1" # 默认

    DEEPSEEK_GENERAL_MODEL="deepseek-chat"
    DEEPSEEK_REASONING_MODEL="deepseek-coder"

    LOG_LEVEL="INFO"
    DEFAULT_PRIORITY="medium"
    PROJECT_NAME="My Task Manager Project"
    DEBUG=False
    TASKS_JSON_PATH="tasks.json" # 任务的默认位置
    TASKS_DIR_PATH="tasks" # 目前未使用，但已定义
    AI_TEMPERATURE=0.7
    AI_MAX_TOKENS=8192
    ```

## 使用方法

所有命令都应从激活虚拟环境的项目根目录运行。主入口点是 `src/cli/main.py`。

你可以使用 `python -m src/cli/main <命令> [选项]` 运行命令。

**常用命令:**

*   **列出任务:**
    ```bash
    python -m src/cli/main list
    python -m src/cli/main list --status pending
    python -m src/cli/main list --priority high
    ```

*   **显示任务详情:**
    ```bash
    python -m src/cli/main show 1      # 显示任务 1
    python -m src/cli/main show 3.1    # 显示任务 3 的子任务 1
    ```
    *(这也会显示估计的复杂度)*

*   **设置任务状态:**
    ```bash
    python -m src/cli/main set-status --id 2 --status done
    python -m src/cli/main set-status --id 1 --id 3.1 --status in-progress
    ```

*   **查找下一个任务:**
    ```bash
    python -m src/cli/main next
    ```

*   **检查依赖:**
    ```bash
    python -m src/cli/main check-deps
    ```

*   **生成占位符文件:**
    ```bash
    python -m src/cli/main generate-file 1
    python -m src/cli/main generate-file 2 --output-dir ./code_files
    ```

**AI 命令:**

*   **解析 PRD:**
    ```bash
    # 首先创建一个 my_project.prd 文件
    python -m src/cli/main parse-prd my_project.prd
    ```

*   **扩展任务:**
    ```bash
    python -m src/cli/main expand 3 # 将任务 3 扩展为子任务
    ```

## 架构概述

*   **`src/config`:** 使用 Pydantic 处理从 `.env` 加载设置。
*   **`src/task_manager`:** 包含核心逻辑:
    *   `data_models.py`: 任务、子任务等的 Pydantic 模型。
    *   `storage.py`: 用于从/向 JSON 加载/保存任务的函数。
    *   `core.py`: 核心任务操作函数 (获取、设置状态、查找下一个)。
    *   `dependencies.py`: 循环依赖检测。
    *   `complexity.py`: 基本复杂度估计。
    *   `file_generator.py`: 占位符文件生成。
    *   `parsing.py`: 使用 AI 解析 PRD 的逻辑。
    *   `expansion.py`: 使用 AI 扩展任务的逻辑。
*   **`src/ai`:** 处理 AI 交互:
    *   `client.py`: 调用 DeepSeek API (通过 OpenAI 客户端) 的核心函数，具有重试机制。
    *   `prompts.py`: 定义用于 AI 生成的提示。
*   **`src/cli`:** 命令行界面:
    *   `main.py`: 定义 Typer 命令并协调对其他模块的调用。
    *   `ui.py`: 使用 `rich` 库显示格式化的输出 (表格、详细信息)。
*   **`tests`:** 包含使用 `pytest` 的单元测试和集成测试。
*   **`tasks.json`:** 用于存储任务数据的默认文件。

## 贡献

欢迎贡献! 请遵循标准编码规范并确保测试通过。

要运行测试:

```bash
uv run pytest tests/unit
uv run pytest tests/integration
```

*(如果需要，请在此处添加更多详细信息: 分支策略、PR 流程等。)*