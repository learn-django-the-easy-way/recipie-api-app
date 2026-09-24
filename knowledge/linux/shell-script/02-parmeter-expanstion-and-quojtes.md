# Parameter expansion to evalute Variables, Quotes

## 1. Parameter Expansion (`${}`)

**Parameter expansion** is used to evaluate and retrieve the value of a variable. While `$var` and `${var}` perform the same basic function, the curly braces provide boundary protection and unlock advanced features.

### A. Boundary Protection & Disambiguation

Explicitly defines where a variable name ends and adjacent text begins.

```bash
drink="tea"
echo "It's $drinktime"    # Outputs: "It's " (Looks for empty '$drinktime')
echo "It's ${drink}time"  # Outputs: "It's teatime"
```

### B. Multi-Digit Positional Parameters

Required when referencing script or function arguments beyond the 9th parameter.

- `$1` through `$9` work normally.
- Use `${10}`, `${11}`, etc., for double digits (using `$10` evaluates `$1` and appends a literal `0`).

### C. `sh` (POSIX) vs. `bash` Compatibility

Basic features are universally supported in standard POSIX `sh`, but advanced manipulation varies:

- **Supported in `sh` & `bash`:**
  - `${var:-default}` — Uses fallback value if unset or empty.
  - `${#var}` — Returns string length.
  - `${var#pattern}` / `${var%pattern}` — Trims matching prefixes/suffixes.
- **Supported ONLY in `bash` / `zsh` (Throws syntax errors in `sh`):**
  - `${var/search/replace}` — Search and replace.
  - `${var:offset:length}` — Substring extraction.
  - `${var^^}` / `${var,,}` — Case conversion.

---

## 2. Quotes: Single (`'`) vs. Double (`"`)

Quotes control how strictly the shell prevents the processing of special characters.

| Feature                             | Single Quotes (`'...'`)    | Double Quotes (`"..."`)          |
| :---------------------------------- | :------------------------- | :------------------------------- |
| **Variable Expansion (`$var`)**     | 🚫 Treated as literal text | Evaluated & Substituted          |
| **Command Substitution (`$(...)`)** | 🚫 Treated as literal text | Evaluated & Executed             |
| **Word Splitting (Spaces)**         | Preserved as one argument  | Preserved as one argument        |
| **Best Used For**                   | Raw text, regex patterns   | Strings with variables or spaces |

### Example Comparison

```bash
name="Alice"
echo 'Hello $name, today is $(date)' # Outputs: Hello $name, today is $(date)
echo "Hello $name, today is $(date)" # Outputs: Hello Alice, today is [current date]
```

### ⚠️ The Golden Rule of Quoting

**Always wrap variable expansions in double quotes** when passing them to commands to prevent whitespace splitting from breaking your logic.

```bash
folder="My Documents"
mkdir $folder    # BAD: Creates two folders ("My" and "Documents")
mkdir "$folder"  # GOOD: Creates one folder ("My Documents")
```

---

## 3. The `unset` Command

The `unset` built-in command deletes variables or functions from the current shell's active memory.

### Usage

- **Variables:** `unset variable_name`
- **Functions:** `unset -f function_name`

### Rules & Behaviors

- **Read-only variables** cannot be unset and will throw an error if attempted.
- **Environment variables** are only deleted for the current active shell session and its child processes.
- Operating on a non-existent variable fails silently with a success exit status (`0`).

---

## 4. Quick Reference: `${}` vs. `$()`

- **`${variable}`** -> Extracts or manipulates a **variable**.
- **`$(command)`** -> Executes a **command** and captures its output (Command Substitution).
