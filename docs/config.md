# config.py

## Overview

This file manages configuration settings, global state variables, constants, and the definition of tools available to the Language Model (LLM) for the Cerebras Assistant application. It loads settings from an external `config.json` file and provides fallbacks for essential parameters.

## Key Components

### Global State and OS Information
- **`os_info: Dict[str, Any]`**
  - **Description:** A dictionary populated at startup with information about the operating system, Python version, and detected shell availability.
- **`base_dir: Path`**
  - **Description:** Represents the current base directory for file operations. Initialized to `Path.cwd()` and can be modified by the user at runtime.
- **`git_context: Dict[str, Any]`**
  - **Description:** A dictionary holding the state for Git interactions, such as whether Git is enabled, the current branch, and if staging should be skipped.
- **`model_context: Dict[str, Any]`**
  - **Description:** Stores the current AI model being used (e.g., default chat model or reasoner model) and a flag indicating if the reasoner is active.
- **`security_context: Dict[str, Any]`**
  - **Description:** Contains security-related settings, primarily flags for requiring user confirmation before executing shell commands (Bash/PowerShell).

### Configuration Loading
- **`load_config() -> Dict[str, Any]`**
  - **Description:** Loads application settings from `config.json` located in the same directory as `config.py`.
  - **Returns:** A dictionary containing the loaded configuration, or an empty dictionary if the file is not found or is invalid.
- **`config: Dict[str, Any]`**
  - **Description:** The dictionary holding the loaded configuration from `config.json`.

### Constants
- **Command Prefixes (`ADD_COMMAND_PREFIX`, `COMMIT_COMMAND_PREFIX`, etc.)**
  - **Description:** String constants defining the prefixes for user-invoked slash commands.
- **Configurable Constants (e.g., `MAX_FILES_IN_ADD_DIR`, `MIN_FUZZY_SCORE`, `DEFAULT_MODEL`)**
  - **Description:** These constants derive their values from the loaded `config` dictionary, with sensible defaults provided if specific settings are missing in `config.json`. They cover areas like file size limits, fuzzy matching thresholds, conversation history limits, and model names.
- **`MODEL_CONTEXT_LIMITS: Dict[str, int]`**
  - **Description:** A dictionary mapping model names to their conservative context token limits. Used by `get_max_tokens_for_model`.
- **`get_max_tokens_for_model(model_name: str) -> int`**
  - **Description:** Returns the context token limit for a given model name, or a default conservative limit if the model is not in `MODEL_CONTEXT_LIMITS`.
- **`EXCLUDED_FILES: Set[str]`, `EXCLUDED_EXTENSIONS: Set[str]`**
  - **Description:** Sets of filenames and file extensions that should be ignored during operations like adding a directory's contents to the conversation context. Loaded from `config.json` with defaults.

### System Prompt
- **`load_system_prompt() -> str`**
  - **Description:** Loads the main system prompt from `system_prompt.txt`. This prompt defines the AI's persona, capabilities, and guidelines.
  - **Returns:** The content of `system_prompt.txt` or a `DEFAULT_SYSTEM_PROMPT` if the file is not found.
- **`DEFAULT_SYSTEM_PROMPT: str`**
  - **Description:** A fallback system prompt string, formatted with current OS information.
- **`SYSTEM_PROMPT: str`**
  - **Description:** The actual system prompt string used by the application, loaded by `load_system_prompt()`.

### Fuzzy Matching Availability
- **`FUZZY_AVAILABLE: bool`**
  - **Description:** A boolean flag that is set to `True` if `thefuzz` library (for fuzzy string matching) is successfully imported, `False` otherwise.

### Function Calling Tools Definition
- **`tools: List[Dict[str, Any]]`**
  - **Description:** A list of dictionaries defining the functions (tools) that the LLM can request to call. Each definition follows the OpenAI function calling specification, including the function name, description, and parameter schema. This list includes tools for file operations (`read_file`, `create_file`, `edit_file`), Git operations (`git_init`, `git_add`, `git_commit`, `git_create_branch`, `git_status`), and system shell commands (`run_powershell`, `run_bash`).

## Important Variables/Constants

This file primarily defines constants and configuration-derived variables. The most critical ones are:
- **`config`**: The dictionary holding all settings from `config.json`.
- **`os_info`**: Provides runtime environment details.
- **`SYSTEM_PROMPT`**: Defines the AI's behavior and capabilities.
- **`DEFAULT_MODEL`, `REASONER_MODEL`**: Specify the AI models to be used.
- **`tools`**: The schema defining functions callable by the LLM.
- **`*_CONTEXT_LIMITS`, `MAX_HISTORY_MESSAGES`**: Control conversation context size.
- **`EXCLUDED_FILES`, `EXCLUDED_EXTENSIONS`**: Define what files to ignore.
- **`security_context`**: Governs whether shell command execution requires user confirmation.

## Usage Examples

This file is not executed directly but is imported by other modules, primarily `main.py` and `utils.py`.

`main.py` imports various constants and contexts:
```python
# In main.py
from config import (
    os_info, base_dir, git_context, model_context, security_context,
    ADD_COMMAND_PREFIX, COMMIT_COMMAND_PREFIX, GIT_BRANCH_COMMAND_PREFIX,
    FUZZY_AVAILABLE, DEFAULT_MODEL, REASONER_MODEL, tools, SYSTEM_PROMPT,
    MAX_FILES_IN_ADD_DIR, MAX_FILE_CONTENT_SIZE_CREATE, EXCLUDED_FILES, EXCLUDED_EXTENSIONS,
    MAX_MULTIPLE_READ_SIZE, config
)

# Accessing a configuration value
print(f"Default model is: {DEFAULT_MODEL}")
# Using the tools definition for an API call
# response = client.chat.completions.create(..., tools=tools, ...)
```

## Dependencies and Interactions

- **Internal Dependencies:**
  - None directly, but it provides configurations used by most other Python files in the project.
- **External Libraries:**
  - `platform`: Used to gather OS and Python version information.
  - `pathlib`: For path manipulations.
  - `json`: For loading `config.json`.
  - `thefuzz` (optional): If installed, enables fuzzy matching capabilities. `FUZZY_AVAILABLE` reflects its presence.
- **Interactions:**
  - Reads `config.json` at startup to load settings.
  - Reads `system_prompt.txt` at startup.
  - Provides constants and context dictionaries that are used throughout the application to control behavior, define LLM capabilities, and manage state.
  - The structure of the `tools` variable is critical for the LLM's function-calling mechanism.
```
