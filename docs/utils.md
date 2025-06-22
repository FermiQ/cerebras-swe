# utils.py

## Overview

This file provides a collection of utility functions supporting the main operations of the Cerebras Assistant. These include functions for detecting system capabilities, managing conversation context (token estimation, truncation), validating tool calls, path normalization, file handling (reading, checking for binary), fuzzy matching for file names and content editing, executing shell commands (Bash/PowerShell), and generating directory tree summaries.

## Key Components

### Shell and System Utilities
- **`detect_available_shells() -> None`**
  - **Description:** Detects which command-line shells (Bash, PowerShell, Zsh, Cmd) are available on the system and updates the `os_info` dictionary in `config.py`.
- **`get_directory_tree_summary(root_dir: Path, max_depth: int = 3, max_entries: int = 100) -> str`**
  - **Description:** Generates a string representation of a directory tree up to a specified depth and number of entries, excluding hidden files/directories.
  - **Returns:** A string formatted as a directory tree.

### Context Management
- **`estimate_token_usage(conversation_history: List[Dict[str, Any]]) -> Tuple[int, Dict[str, int]]`**
  - **Description:** Estimates the number of tokens used by the current conversation history based on character count and message structure.
  - **Returns:** A tuple containing the total estimated tokens and a dictionary breaking down token usage by role (system, user, assistant, tool).
- **`get_context_usage_info(conversation_history: List[Dict[str, Any]], model_name: str = None) -> Dict[str, Any]`**
  - **Description:** Provides a comprehensive dictionary of context usage statistics, including total messages, estimated tokens, max tokens for the current model, usage percentage, file context count, and flags for approaching or critical context limits.
  - **Returns:** A dictionary with context usage details.
- **`smart_truncate_history(conversation_history: List[Dict[str, Any]], max_messages: int = MAX_HISTORY_MESSAGES, model_name: str = None) -> List[Dict[str, Any]]`**
  - **Description:** Intelligently truncates the conversation history to stay within token limits. It prioritizes keeping the main system prompt, recent file contexts (smallest and newest first), and recent conversational turns, especially preserving tool call sequences (assistant request + tool responses).
  - **Returns:** The truncated list of conversation messages.

### Tool Call and Prompt Utilities
- **`validate_tool_calls(accumulated_tool_calls: List[Dict[str, Any]]) -> List[Dict[str, Any]]`**
  - **Description:** Validates a list of tool calls (e.g., from an LLM response) to ensure they have necessary fields (ID, function name) and valid JSON arguments.
  - **Returns:** A list containing only the valid tool calls.
- **`get_model_indicator() -> str`**
  - **Description:** Returns an emoji indicating whether the current model is the reasoner (🧠) or the chat model (💬).
- **`get_prompt_indicator(conversation_history: List[Dict[str, Any]], model_name: str = None) -> str`**
  - **Description:** Generates a string for the command prompt, including the model indicator, current Git branch (if applicable), and a context health indicator (🔴, 🟡, 🔵).

### File and Path Utilities
- **`normalize_path(path_str: str, allow_outside_project: bool = False) -> str`**
  - **Description:** Normalizes a given path string, resolving it to an absolute path. By default, it enforces that the path stays within the project's `base_dir` for security.
  - **Returns:** The normalized absolute path string.
  - **Raises:** `ValueError` if the path is outside the project directory and `allow_outside_project` is `False`.
- **`is_binary_file(file_path: str, peek_size: int = 1024) -> bool`**
  - **Description:** Checks if a file is likely binary by looking for null bytes within the first `peek_size` bytes.
  - **Returns:** `True` if the file appears to be binary, `False` otherwise.
- **`read_local_file(file_path: str) -> str`**
  - **Description:** Reads the content of a local file, assuming UTF-8 encoding.
  - **Returns:** The file content as a string.
- **`add_file_context_smartly(conversation_history: List[Dict[str, Any]], file_path: str, content: str, max_context_files: int = MAX_CONTEXT_FILES) -> bool`**
  - **Description:** Adds file content to the conversation history as a system message. It manages the number of active file contexts (removing older or larger ones if limits are hit), avoids duplicates, and tries to insert the context before the last user message. It also defers adding context if the assistant has pending tool calls.
  - **Returns:** `True` if the file context was added or deferred successfully, `False` if rejected (e.g., due to size).

### Fuzzy Matching Utilities (conditional on `thefuzz` library)
- **`find_best_matching_file(root_dir: Path, user_path: str, min_score: int = MIN_FUZZY_SCORE) -> Optional[str]`**
  - **Description:** Searches for the best file match for a given `user_path` within `root_dir` using fuzzy string matching on filenames. Skips hidden and excluded files/directories.
  - **Returns:** The full, resolved path of the best match if the score is above `min_score`, otherwise `None`.
- **`apply_fuzzy_diff_edit(path: str, original_snippet: str, new_snippet: str) -> None`**
  - **Description:** Edits a file by replacing `original_snippet` with `new_snippet`. It first attempts an exact match. If that fails and fuzzy matching is available, it uses fuzzy matching to find the best occurrence of `original_snippet` to replace.
  - **Raises:** `ValueError` if the snippet is not found, the match is ambiguous, or the fuzzy score is too low.

### Shell Command Execution
- **`run_bash_command(command: str) -> Tuple[str, str]`**
  - **Description:** Executes a given command string using the Bash shell. It includes logic to find a suitable Bash executable (system bash, WSL bash, Git Bash on Windows).
  - **Returns:** A tuple containing the standard output and standard error of the command.
- **`run_powershell_command(command: str) -> Tuple[str, str]`**
  - **Description:** Executes a given command string using PowerShell. It attempts to use `powershell` (Windows PowerShell) or `pwsh` (PowerShell Core) based on availability.
  - **Returns:** A tuple containing the standard output and standard error of the command.

## Important Variables/Constants

- **`console = Console()`**
  - **Description:** An instance of `rich.console.Console` used for formatted output to the terminal.
- Imports various constants from `config.py` (e.g., `MIN_FUZZY_SCORE`, `MAX_HISTORY_MESSAGES`, `EXCLUDED_FILES`, `EXCLUDED_EXTENSIONS`) to control the behavior of utility functions.
- **`FUZZY_AVAILABLE: bool`** (from `config.py`)
  - **Description:** Determines if fuzzy matching functions can be used.

## Usage Examples

These functions are typically called by `main.py` or other parts of the application.

```python
# In main.py, adding a file to context:
from utils import normalize_path, read_local_file, add_file_context_smartly
# ...
normalized_path = normalize_path(user_provided_path)
content = read_local_file(normalized_path)
add_file_context_smartly(conversation_history, normalized_path, content)

# In main.py, executing an LLM tool call for a bash command:
from utils import run_bash_command
# ...
output, error = run_bash_command("ls -la")
# result = f"Bash Output:\n{output}\nBash Error:\n{error}"

# In main.py, truncating history:
from utils import smart_truncate_history
# ...
conversation_history = smart_truncate_history(conversation_history, model_name=current_model)
```

## Dependencies and Interactions

- **Internal Dependencies:**
  - `config.py`: Relies heavily on constants and global contexts (like `os_info`, `git_context`, `model_context`) defined in `config.py`.
- **External Libraries:**
  - `rich`: For console output formatting.
  - `platform`: Used by `detect_available_shells` and shell command runners to determine OS specifics.
  - `subprocess`: Used to run external shell commands and detect shell availability.
  - `thefuzz` (optional): If `FUZZY_AVAILABLE` is `True`, this library is used for fuzzy string matching in `find_best_matching_file` and `apply_fuzzy_diff_edit`.
- **Interactions:**
  - Modifies `config.os_info` via `detect_available_shells()`.
  - Interacts with the file system for reading files, listing directories, and checking file properties.
  - Executes external processes for shell commands (`bash`, `powershell`).
  - Many functions take `conversation_history` as input to make decisions based on its current state (e.g., for truncation or context health indicators).
```
