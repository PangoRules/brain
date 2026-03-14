# Chapter 33 – Flow Control: Looping with `for`

## Quick Reference

| Form | Use |
|------|-----|
| `for var in list; do ... done` | Iterate over words |
| `for var in {1..10}; do` | Brace expansion range |
| `for var in *.txt; do` | Glob/pathname expansion |
| `for var in $(cmd); do` | Command substitution list |
| `for var; do` | Iterate over positional params (`$@`) |
| `for (( i=0; i<N; i++ )); do` | C-style numeric loop |

---

## Traditional Shell Form

```bash
for variable [in words]; do
    commands
done
```

### Word sources

```bash
# Literal list
for i in A B C D; do echo "$i"; done

# Brace expansion
for i in {1..5}; do echo "$i"; done
for i in {0..23}; do arr[i]=0; done       # init array
for i in {A..Z}; do echo "$i"; done

# Pathname expansion (glob)
for f in *.log; do
    [[ -e "$f" ]] || continue              # guard against no-match
    gzip "$f"
done

# Command substitution (intentional word splitting — no quotes)
for word in $(cat /etc/hosts); do
    echo "$word"
done
```

### Omitting `in words` — iterate over positional params

```bash
# Equivalent to: for i in "$@"; do
for i; do
    [[ -r "$i" ]] || continue
    process_file "$i"
done
```

---

## C-Style Form

```bash
for (( expr1; expr2; expr3 )); do
    commands
done
```

- `expr1` — initialization (runs once)
- `expr2` — condition (checked before each iteration)
- `expr3` — increment (runs after each iteration)

```bash
for (( i=0; i<5; i++ )); do
    echo "$i"
done

# Count down
for (( i=10; i>=0; i-- )); do
    printf "\r%d " "$i"
    sleep 1
done; echo

# Two variables
for (( i=0, j=10; i<10; i++, j-- )); do
    echo "$i $j"
done
```

---

## Practical Patterns

```bash
# Process files passed as arguments
for file in "$@"; do
    [[ -f "$file" ]] || { echo "Not a file: $file" >&2; continue; }
    echo "Processing: $file"
done

# Batch rename (e.g. lowercase all .JPG → .jpg)
for f in *.JPG; do
    [[ -e "$f" ]] || continue
    mv "$f" "${f%.JPG}.jpg"
done

# Run N parallel jobs (on your 24-thread i9)
jobs=0
for f in *.mkv; do
    ffmpeg -i "$f" "${f%.mkv}.mp4" &
    (( ++jobs ))
    (( jobs >= 24 )) && { wait; jobs=0; }
done
wait

# Collect stats per hour (array + C-style loop display)
for (( i=0; i<=23; i++ )); do
    printf "%02d: %d files\n" "$i" "${hour_count[i]}"
done

# Retry loop with counter
for (( attempt=1; attempt<=3; attempt++ )); do
    command_that_might_fail && break
    echo "Attempt $attempt failed, retrying..."
    sleep 2
done

# Modulo — do something every N iterations
for (( i=0; i<=100; i++ )); do
    (( i % 10 == 0 )) && echo "--- checkpoint $i ---"
    process "$i"
done
```

---

## `for` vs `while` — When to Use Each

| Situation | Use |
|-----------|-----|
| Known list of items (files, args, words) | `for var in list` |
| Numeric range or counter | `for (( ))` |
| Loop until condition changes | `while` / `until` |
| Reading lines from file/stdin | `while IFS= read -r` |
| Unknown number of iterations | `while` |

---

## Notes

- **Glob no-match guard**: when using `for f in *.ext`, add `[[ -e "$f" ]] || continue` — if no files match, the literal `*.ext` string becomes the loop variable
- **Word splitting in command substitution**: `for i in $(cmd)` intentionally word-splits; this is one of the few cases you don't quote `$(cmd)`
- **zsh**: `for` works identically; zsh also supports `for f (*.txt) { echo $f }` shorthand
- **Parallelism**: use `&` + `wait` inside `for` loops to exploit your 24-thread i9 for batch processing
