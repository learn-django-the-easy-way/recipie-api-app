# 🐧 Linux Shell Scripting & Environment

A structured guide covering variables, execution context, pathing, and conditional testing blocks in modern Linux shell environments (Bash/Zsh).

---

## 1. The `export` Command

The `export` command is a built-in shell utility used to **turn local shell variables into global environment variables**.

- **Default Behavior:** Variables created in a terminal session are private to that specific shell process. Child processes, subshells, and external scripts cannot see them.
- **Export Behavior:** Marking a variable with `export` instructs the shell to pass that variable down into the environment of **all child processes and scripts** spawned from that session.

### 💻 Code Examples

```bash
# WITHOUT EXPORT (Local Variable)
MY_VAR="Hello"
bash -c 'echo $MY_VAR'  # Output is blank (child process cannot access it)

# WITH EXPORT (Environment Variable)
export MY_VAR="Hello"
bash -c 'echo $MY_VAR'  # Output: Hello
```

### 🛠️ Common Options

- `export -p`: Lists all environment variables exported in the current shell session.
- `export -n <var>`: Removes the export attribute, turning it back into a local variable.
- `export -f <func>`: Exports a Bash function to child processes.

---

## 2. Script Execution Context: `./script.sh` / `bash script.sh` vs `source script.sh` / `. script.sh`

How you execute a script changes how it interacts with local and environment variables.

### 🔄 The Mechanics

- **`./script.sh` (or `bash script.sh`)**: Spawns a completely new **child process** (subshell). It can only see variables that were explicitly `export`ed by the parent shell.
- **`source script.sh` (or `. script.sh`)**: Tells the current shell to read and run the script **directly inside the active process**. Because no child process is created, it natively reads all local variables.

### ⚠️ A Critical Caveat

Even when using `source`, if your script invokes **external binaries** (e.g., executing a Python script via `python3 main.py` or running background binaries), those external tools run as child processes. Therefore, they still require variables to be **`export`ed** to read them.

---

## 3. Environment Variables vs. The `PATH` Variable

Understanding global system configurations versus execution boundaries.

- **Environment Variables (General):** Global key-value pairs (`USER=ubuntu`, `LANG=en_US.UTF-8`) that pass general configuration settings and metadata down to applications.
- **The `PATH` Variable (Specific):** A specific, highly critical environment variable containing a colon-separated list of directories (`/usr/bin:/bin:/usr/local/bin`).

When you type a command (like `ls` or `python`), the system searches these `PATH` directories sequentially from left to right to find and execute the binary file. Breaking your `PATH` variable results in a system-wide `command not found` error for basic tools.

---

## 4. Managing Environment Variable Persistence

Where you save a variable determines its lifetime and accessibility.

1. **Temporary Session:** `export MY_VAR="value"` (Disappears when the terminal window closes).
2. **Permanent Per-User:** Added to `~/.bashrc` or `~/.zshrc`. (Loaded every time that specific user opens a shell session).
3. **Permanent System-Wide:** Added to `/etc/environment` **without** the `export` keyword. (Loaded for all users and background services upon system boot).

---

## 5. Conditional Blocks: Single `[` vs. Double `[[` Brackets

Choosing the right structural block for evaluating logic strings in scripts.

- **`[` (Single Bracket):** An alias for the legacy external program `/usr/bin/test`. It handles data inputs strictly as raw text arguments, making it highly fragile. It will crash or throw syntax errors if a variable is empty or contains spaces unless strings are explicitly wrapped in double quotes (`"$VAR"`).
- **`[[` (Double Bracket):** A modern built-in shell keyword supported by Bash, Zsh, and Ksh. Because it is natively understood by the shell parser, it includes robust safety guards that handle spaces and empty variables perfectly without breaking.

### 📊 Feature Mapping Matrix

| Boundary Feature           | Single Brackets `[ ... ]`                   | Double Brackets `[[ ... ]]`                    |
| :------------------------- | :------------------------------------------ | :--------------------------------------------- |
| **Architecture**           | External Program (`test`)                   | Internal Shell Keyword                         |
| **Portability**            | High (Strict POSIX / Works on `sh`)         | Bash, Zsh, and Ksh only                        |
| **Unquoted Spaces**        | ❌ Crashes (`too many arguments`)           | Safely parsed as a single string               |
| **Logical Join Operators** | Requires legacy flags `-a` (AND), `-o` (OR) | Uses standard operators `&&` and `             |
| **Pattern Matching**       | ❌ Unsupported                              | Supports wildcard expressions (e.g., `*[Yy]*`) |
| **Regex Parsing**          | ❌ Unsupported                              | Natively supported via the `=~` operator       |

### 💡 Rule of Thumb

Always default to **`[[ ... ]]`** if your script header targets modern shells (`#!/bin/bash` or `#!/bin/zsh`). Reserve **`[ ... ]`** exclusively for hyper-minimal, strict POSIX compliance scripts targeting bare-bones environments like alpine containers or embedded systems (`#!/bin/sh`).
