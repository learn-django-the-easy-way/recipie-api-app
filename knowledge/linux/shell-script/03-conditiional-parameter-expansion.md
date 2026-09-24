# 📖 Conditional Parameter Expansion

This reference guide summarizes conditional parameter expansions in Bash shell scripting. It covers shortcuts used to check, fallback, assign, or crash based on a variable's status.

---

## 🌟 1. Core Definitions: Set, Unset, and Empty

Understanding the exact status of a variable in memory is critical for predicting how parameter expansions behave.

| State              | Analogy                                                       | Code Example                                            |
| :----------------- | :------------------------------------------------------------ | :------------------------------------------------------ |
| **Unset**          | The "box" **does not exist** at all in memory.                | _(Variable was never typed, or `unset var` was called)_ |
| **Empty (Null)**   | The "box" **exists**, but it contains **absolutely nothing**. | `var=""` or `var=`                                      |
| **Set (Non-Null)** | The "box" **exists** and has **any characters** inside it.    | `var="hello"`, `var=0`, `var="false"`, `var="   "`      |

---

## ⚡ 2. Quick Reference: The 4 Main Modifiers (With Colon)

When using the **colon (`:`)**, Bash checks if the variable is **either unset OR empty**.

| Syntax                  | English Meaning              | What it returns if `var` is **Set** | What it returns if `var` is **Empty or Unset** | Modifies `var` permanently? |
| :---------------------- | :--------------------------- | :---------------------------------- | :--------------------------------------------- | :-------------------------- |
| **`${var:+alternate}`** | _"Use alternate if exists"_  | `alternate`                         | _Nothing_ (Empty string)                       | ❌ **No** (Temporary)       |
| **`${var:-default}`**   | _"Use default fallback"_     | Actual value of `var`               | `default`                                      | ❌ **No** (Temporary)       |
| **`${var:=value}`**     | _"Save default permanently"_ | Actual value of `var`               | `value`                                        | **Yes** (Permanent)         |
| **`${var:?error_msg}`** | _"Crash script if missing"_  | Actual value of `var`               | _Crashes script_ with `error_msg`              | ❌ **No** (Destructive)     |

---

## 🔍 3. Detailed Breakdown & Code Examples

### A. `${var:+alternate}` — The "Bonus/Flag" Expansion

- **Behavior:** If the variable is set and not empty, it returns the completely different `alternate` value. Otherwise, it prints nothing.
- **Best Used For:** Conditionally adding optional command-line flags.
- **Code Example:**

  ```bash
  DEBUG="true"
  echo ${DEBUG:+--verbose}   # Outputs: --verbose (Variable is set)
  echo "$DEBUG"              # Outputs: true (The variable never changed!)

  EMPTY_VAR=""
  echo ${EMPTY_VAR:+--verbose} # Outputs nothing (Variable is empty)
  ```

### B. `${var:-default}` — The "Temporary Fallback" Expansion

- **Behavior:** If the variable has a value, use it. If it is empty or missing, use the default string _just for this command_.
- **Best Used For:** Providing quick, safe backups without altering your configuration variables.
- **Code Example:**

  ```bash
  USER_PORT=""
  echo ${USER_PORT:-8080}    # Outputs: 8080 (Borrowed the fallback)
  echo "Port is $USER_PORT"  # Outputs: Port is  (Variable remains empty!)
  ```

### C. `${var:=value}` — The "Permanent Assignment" Expansion

- **Behavior:** Works like `:-`, but if the variable is empty or missing, it **permanently writes** the default value into that variable for the rest of the script.
- **Best Used For:** Initializing default configuration states at the top of a script.
- **Code Example:**

  ```bash
  USER_THEME=""
  echo ${USER_THEME:="dark"} # Outputs: dark
  echo "Theme is $USER_THEME" # Outputs: Theme is dark (Permanently changed!)
  ```

  _(Note: This cannot be used on positional parameters like `$1`, `$2`, etc.)_

### D. `${var:?error_msg}` — The "Safety Net" Expansion

- **Behavior:** If the variable is empty or missing, the script immediately prints the error message to `stderr` and exits (stops running).
- **Best Used For:** Protecting critical paths (like target cleanup directories or passwords) from blank values.
- **Code Example:**

  ```bash
  DB_PASSWORD=""
  # Script terminates instantly with the designated error message
  BACKUP_PASS=${DB_PASSWORD:?"Error: DB_PASSWORD is required!"}
  ```

---

## ⚠️ 4. Dropping the Colon (`:`) — Checking Existence Only

You can drop the colon from any of these expressions (e.g., `${var-default}`). Doing so fundamentally alters the expansion logic:

- **With Colon (`:`) Rules:** Triggers if the variable is **unset OR empty (`""`)**.
- **Without Colon Rules:** Triggers **ONLY if the variable is completely unset**. If the variable exists but is blank, it leaves it alone.

### How Dropping the Colon Affects an Empty Variable (`var=""`)

| Syntax  | With Colon (`:`) Behavior | Without Colon Behavior                                |
| :------ | :------------------------ | :---------------------------------------------------- |
| **`-`** | Returns default           | Returns **empty string** (sees the blank box)         |
| **`=`** | Assigns & returns default | Returns **empty string** (does not overwrite)         |
| **`+`** | Returns **empty string**  | Returns the **alternate value** (sees the box exists) |
| **`?`** | **Crashes** script        | Returns **empty string** (script passes safely)       |

### Comparative Example (`:-` vs `-`)

```bash
NAME="" # The variable exists, but it's an empty string

# 1. With Colon: Checks for values. "Is it blank? Yes. Use fallback."
echo ${NAME:-"Guest"}   # Outputs: Guest

# 2. Without Colon: Checks for existence. "Does it exist? Yes. Use its blank value."
echo ${NAME-"Guest"}    # Outputs nothing (empty line)
```

---

## 🎯 Summary Memory Rule

- **Temporary (Doesn't change variable):** `${var:-default}` and `${var:+alternate}`
- **Permanent (Saves to variable):** `${var:=default}`
- **Destructive (Kills the script):** `${var:?error}`
- **The Colon (`:`):** Include it to catch blank strings (`""`). Drop it if blank strings are allowed.
