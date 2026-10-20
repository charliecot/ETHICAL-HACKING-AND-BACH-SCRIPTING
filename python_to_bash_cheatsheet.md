# Python-to-Bash Cheatsheet

A quick reference for translating common Python concepts into Bash.

## 1. Python vs Bash

| Python | Bash | Meaning |
|---|---|---|
| `len(items)` | `${#items[@]}` | Count array items |
| `len(text)` | `${#text}` | Count characters in a string |
| `items[0]` | `${items[0]}` | Access the first array item |
| `items.append("x")` | `items+=("x")` | Append an item to an array |
| `for item in items:` | `for item in "${items[@]}"; do ...; done` | Loop through array items |
| `range(len(items))` | `for ((i=0; i<${#items[@]}; i++)); do ...; done` | Loop through array indexes |
| `print(value)` | `printf '%s\n' "$value"` or `echo "$value"` | Print a value |
| `input()` | `read -r variable` | Read user input |
| `str(value)` | `printf '%s' "$value"` | Print a value as text |
| `type(value)` | `declare -p value` | Inspect a Bash variable's declaration |

## 2. Random: Python vs Bash

| Python | Bash | Meaning |
|---|---|---|
| `random.randint(1, 10)` | `shuf -i 1-10 -n 1` | Random integer from 1 to 10 |
| `random.randrange(10)` | `shuf -i 0-9 -n 1` | Random integer from 0 to 9 |
| `random.choice(items)` | `items[$((RANDOM % ${#items[@]}))]` | Select a random array item |
| `random.sample(items, 2)` | `printf '%s\n' "${items[@]}" \| shuf -n 2` | Select two items without replacement |
| `random.shuffle(items)` | `printf '%s\n' "${items[@]}" \| shuf` | Output items in random order |
| `random.choice([True, False])` | `((RANDOM % 2)) && echo True \|\| echo False` | Produce a random True/False-like result |
| `random.random()` | `awk 'BEGIN{srand(); print rand()}'` | Floating-point value from 0 up to 1 |
| `random.randint(1, 6)` | `shuf -i 1-6 -n 1` | Simulate a die roll |
| `random.choice(["Heads", "Tails"])` | `((RANDOM % 2)) && echo Heads \|\| echo Tails` | Flip a coin |
| `random.seed(42)` | `RANDOM=42` | Seed Bash's `$RANDOM` generator; behavior differs from Python |

> **Tip:** Bash's `$RANDOM` is a built-in variable that produces integers from `0` to `32767`. For random integer ranges, `shuf -i MIN-MAX -n 1` is often easier to read and avoids modulo bias.

## 3. Understanding Bash array syntax

```bash
items=("Python" "Bash" "JavaScript")

# Number of items in the array
echo "${#items[@]}"       # 3

# First item (indexes start at 0)
echo "${items[0]}"        # Python

# Number of characters in the first item
echo "${#items[0]}"       # 6

# Append a new item
items+=("Django")

# Print all items
printf '%s\n' "${items[@]}"
```

Remember:

- `${#items[@]}` counts array items.
- `${#text}` counts characters in a string.
- `${items[0]}` gets the item at index `0`.
- `"${items[@]}"` expands to all array elements while preserving spaces within each element.

## 4. Random selection from a Bash array

```bash
#!/usr/bin/env bash

items=("Python" "Bash" "JavaScript" "Django")

# Check that the array is not empty before selecting
if ((${#items[@]} > 0)); then
    index=$((RANDOM % ${#items[@]}))
    printf 'Random item: %s\n' "${items[$index]}"
fi
```

For a more even random selection, use `shuf`:

```bash
items=("Python" "Bash" "JavaScript" "Django")

# Print one randomly selected item
printf '%s\n' "${items[@]}" | shuf -n 1

# Print two different randomly selected items
printf '%s\n' "${items[@]}" | shuf -n 2

# Print all items in random order
printf '%s\n' "${items[@]}" | shuf
```

`shuf` is a Linux command, not a Bash built-in. It is normally available on Ubuntu.

## 5. Random number examples

```bash
# Built-in Bash random integer: 0–32767
echo "$RANDOM"

# Quick ranges using Bash arithmetic
echo "$((RANDOM % 10))"       # 0–9
echo "$((RANDOM % 10 + 1))"   # 1–10
echo "$((RANDOM % 6 + 1))"    # 1–6 (dice)

# Use shuf for an inclusive integer range
shuf -i 1-100 -n 1            # One number from 1–100
shuf -i 1-100 -n 5            # Five numbers from 1–100, no repeats

# A floating-point random value from 0 up to 1
awk 'BEGIN{srand(); print rand()}'
```

The `$RANDOM % N` pattern can have a small statistical bias when the possible values do not divide evenly into `N`. Use `shuf` when you want a straightforward random integer range.

## 6. Complete practice script

Save this as `python_to_bash.sh`:

```bash
#!/usr/bin/env bash

# Create an array
items=("Python" "Bash" "JavaScript")

# Count the array items
printf 'Number of items: %s\n' "${#items[@]}"

# Count characters in a string
text="Hello"
printf 'String length: %s\n' "${#text}"

# Access the first item
printf 'First item: %s\n' "${items[0]}"

# Append an item
items+=("Django")

# Select a random item
index=$((RANDOM % ${#items[@]}))
printf 'Random item: %s\n' "${items[$index]}"

# Generate a random number from 1 to 10
printf 'Random number: %s\n' "$((RANDOM % 10 + 1))"

# Loop through all items
for item in "${items[@]}"; do
    printf 'Learning: %s\n' "$item"
done

# Show the items in random order (requires shuf)
printf '\nShuffled items:\n'
printf '%s\n' "${items[@]}" | shuf
```

Run it on Ubuntu or WSL:

```bash
chmod +x python_to_bash.sh
./python_to_bash.sh
```

## 7. Quick memory rules

- Python `len(items)` → Bash `${#items[@]}`
- Python `len(text)` → Bash `${#text}`
- Python `items.append("x")` → Bash `items+=("x")`
- Python `random.randint(a, b)` → Bash/Linux `shuf -i a-b -n 1`
- Python `random.choice(items)` → choose a random valid array index, or use `shuf`
- Python `print(value)` → Bash `printf '%s\n' "$value"`
- Python `input()` → Bash `read -r variable`
