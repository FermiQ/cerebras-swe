# main.py

## Overview

This file is the main entry point and core logic for the Cerebras Assistant application. It handles user commands, manages the conversation flow with the AI model, and integrates various functionalities like file operations, Git interactions, and system information display.

## Key Components

### Pydantic Models
- **`FileToCreate(BaseModel)`**
  - **Description:** Defines the structure for creating a new file, including its path and content.
  - **Attributes:**
    - `path: str`: The path where the file will be created.
    - `content: str`: The content to be written to the file.
- **`FileToEdit(BaseModel)`**
  - **Description:** Defines the structure for editing an existing file, specifying the original snippet to find and the new snippet to replace it with.
  - **Attributes:**
    - `path: str`: The path of the file to be edited.
    - `original_snippet: str`: The existing content snippet to be replaced.
    - `new_snippet: str`: The new content snippet to insert.

### File Operations
- **`create_file(path: str, content: str, require_confirmation: bool = True) -> None`**
  - **Description:** Creates or overwrites a file with the given content. It includes checks for file size limits, path validity, and prompts for confirmation when overwriting existing files.
  - **Parameters:**
    - `path: str`: The file path.
    - `content: str`: The content to write to the file.
    - `require_confirmation: bool`: If True (default), asks the user before overwriting an existing file.
- **`add_directory_to_conversation(directory_path: str, conversation_history: List[Dict[str, Any]]) -> None`**
  - **Description:** Scans a directory and adds the content of its valid files to the conversation history for context. It skips dotfiles, excluded files/extensions, and binary files.
  - **Parameters:**
    - `directory_path: str`: The path to the directory.
    - `conversation_history: List[Dict[str, Any]]`: The current conversation history list.

### Git Operations
- **`stage_file(file_path_str: str) -> bool`**
  - **Description:** Stages a specified file for a Git commit if Git is enabled and not set to skip staging.
  - **Parameters:**
    - `file_path_str: str`: The path of the file to stage.
  - **Returns:** `bool`: True if staging was successful, False otherwise.
- **`get_git_status_porcelain() -> Tuple[bool, List[Tuple[str, str]]]`**
  - **Description:** Retrieves the Git status in a porcelain format, which is easily parsable.
  - **Returns:** `Tuple[bool, List[Tuple[str, str]]]`: A tuple containing a boolean indicating if there are changes, and a list of tuples with status codes and filenames.
- **`create_gitignore() -> None`**
  - **Description:** Creates a `.gitignore` file with common patterns if one doesn't already exist. Allows users to add custom patterns.
- **`user_commit_changes(message: str) -> bool`**
  - **Description:** Commits staged changes with a user-provided message. It prompts the user if no changes are staged or if there are unstaged changes.
  - **Parameters:**
    - `message: str`: The commit message.
  - **Returns:** `bool`: True if the commit was successful or if the user aborted an action (like staging unstaged changes).

### Command Handlers
- **`show_git_status_cmd() -> bool`**
  - **Description:** Displays the current Git status, including branch information and a table of changed files.
- **`initialize_git_repo_cmd() -> bool`**
  - **Description:** Initializes a new Git repository in the base directory. Prompts to create a `.gitignore` and make an initial commit.
- **`create_git_branch_cmd(branch_name: str) -> bool`**
  - **Description:** Creates a new Git branch and switches to it. If the branch exists, it offers to switch to it.
- **`try_handle_..._command(...)` functions (various)**
  - **Description:** A series of functions (`try_handle_add_command`, `try_handle_git_add_command`, `try_handle_clear_command`, etc.) that parse user input to detect and execute specific slash commands (e.g., `/add`, `/git add`, `/clear`). Each returns `True` if it handled the command, `False` otherwise.

### LLM Tool Handler Functions
- **`ensure_file_in_context(file_path: str, conversation_history: List[Dict[str, Any]]) -> bool`**
  - **Description:** Ensures that the content of a specified file is loaded into the conversation history.
- **`llm_git_init() -> str`**, **`llm_git_add(file_paths: List[str]) -> str`**, etc.
  - **Description:** These functions are designed to be called by the Language Model (LLM) to perform Git operations. They wrap the core Git functionalities and return string results suitable for the LLM.
- **`execute_function_call_dict(tool_call_dict: Dict[str, Any]) -> str`**
  - **Description:** The central dispatcher for executing function calls requested by the LLM. It takes a dictionary representing the tool call, identifies the function, parses arguments, executes it, and returns a string result. This function handles calls like `read_file`, `create_file`, `edit_file`, Git operations, and shell command execution (`run_powershell`, `run_bash`), including security confirmations for shell commands.

### Main Loop & Entry Point
- **`initialize_application() -> None`**
  - **Description:** Initializes the application state, such as detecting available shells and checking for an existing Git repository to set up the initial Git context.
- **`main_loop() -> None`**
  - **Description:** The core interactive loop of the application. It:
    - Initializes conversation history with the system prompt and initial context (directory structure, OS info).
    - Prompts the user for input.
    - Handles user commands (slash commands).
    - Sends user input to the configured AI model (Cerebras API).
    - Manages context size and performs truncation if necessary.
    - Processes the AI's response, including text and tool calls.
    - Executes tool calls via `execute_function_call_dict`.
    - Handles potential multi-turn interactions if the AI makes further tool calls after receiving tool results.
    - Implements error handling and user interruption (Ctrl+C).
- **`main() -> None`**
  - **Description:** The main entry point of the script (`if __name__ == "__main__":`). It prints a welcome panel, shows fuzzy matching and shell status, calls `initialize_application()`, and then starts the `main_loop()`.

## Important Variables/Constants

- **`client = Cerebras(api_key=os.getenv("CEREBRAS_API_KEY"))`**
  - **Description:** The Cerebras SDK client instance used for all interactions with the AI models. API key is loaded from environment variables.
- **`prompt_session = PromptSession(...)`**
  - **Description:** An instance of `prompt_toolkit.PromptSession` used for rich command-line input with history and styling.
- **`conversation_history: List[Dict[str, Any]]`**
  - **Description:** A list of dictionaries that stores the entire conversation with the AI, including system prompts, user messages, assistant responses, and tool call interactions. This is the primary context sent to the AI model.
- **`base_dir: Path`**
  - **Description:** (Imported from `config.py`, but mutable via `/folder` command). Represents the current base directory for all relative file operations. Defaults to the current working directory at startup.
- **`git_context: Dict[str, Any]`**
  - **Description:** (Imported from `config.py`). A dictionary holding the state and configuration for Git interactions, such_as `enabled`, `branch`, `skip_staging`.
- **`model_context: Dict[str, Any]`**
  - **Description:** (Imported from `config.py`). A dictionary holding the state for model selection, like `current_model` and `is_reasoner`.
- **`security_context: Dict[str, Any]`**
  - **Description:** (Imported from `config.py`). A dictionary holding security-related settings, like `require_bash_confirmation`.
- **`tools: List[Dict[str, Any]]`**
  - **Description:** (Imported from `config.py`). A list defining the tools (functions) available to the LLM, formatted according to the OpenAI function calling spec.
- **`SYSTEM_PROMPT: str`**
  - **Description:** (Imported from `config.py`). The initial system prompt that sets the persona and capabilities of the AI assistant.
- **`ADD_COMMAND_PREFIX`, `COMMIT_COMMAND_PREFIX`, etc.**
  - **Description:** (Imported from `config.py`). String constants defining the prefixes for various user commands.

## Usage Examples

The file is executed directly to start the assistant:
```bash
python main.py
```
Once running, users interact via the command line. Examples of interactions:

- **Adding a file to context:** `/add src/my_file.py`
- **Asking a question:** `How do I refactor the login function in auth.py?`
- **Initializing Git:** `/git init`
- **Committing changes:** `/git commit -m "Implemented new feature"` (or `/git commit` then enter message)

The LLM can also request function calls, for example:
```json
{
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "read_file",
        "arguments": "{\"file_path\": \"src/utils.py\"}"
      }
    }
  ]
}
```
This would trigger the `execute_function_call_dict` to run `read_local_file("src/utils.py")`.

## Dependencies and Interactions

- **Internal Dependencies:**
  - `config.py`: For loading configuration variables, constants (like command prefixes, model names, system prompt), and tool definitions.
  - `utils.py`: For utility functions like path normalization, file reading, context management (`smart_truncate_history`, `get_context_usage_info`), shell command execution, and fuzzy matching.
- **External Libraries:**
  - `cerebras.cloud.sdk`: For interacting with the Cerebras AI models.
  - `pydantic`: For data validation and settings management through models (`FileToCreate`, `FileToEdit`).
  - `python-dotenv`: For loading environment variables (e.g., `CEREBRAS_API_KEY`).
  - `rich`: For rich text formatting in the console (panels, tables, styled text).
  - `prompt_toolkit`: For enhanced command-line input.
- **Interactions:**
  - Interacts heavily with the file system for reading, writing, and listing files/directories based on user commands or LLM requests.
  - Interacts with the Git version control system via `subprocess` calls to execute `git` commands.
  - Interacts with the operating system shell (Bash, PowerShell) via `subprocess` for `run_bash` and `run_powershell` tool calls.
  - The core interaction is with the Cerebras API, sending conversation history and receiving model responses (text and tool calls).
```
