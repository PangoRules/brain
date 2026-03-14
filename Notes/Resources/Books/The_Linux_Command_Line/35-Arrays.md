# Chapter 35 – Arrays

## Quick Reference

| Operation | Syntax |
|-----------|--------|
| Create / assign single element | `arr[0]="value"` |
| Assign all at once | `arr=(a b c d)` |
| Assign with explicit indices | `arr=([0]=a [2]=c [5]=f)` |
| Declare explicitly | `declare -a arr` |
| Access element | `${arr[n]}` |
| All elements (separate words) | `"${arr[@]}"` |
| All elements (one string) | `"${arr[*]}"` |
| Number of elements | `${#arr[@]}` |
| Length of element n | `${#arr[n]}` |
| All indices | `"${!arr[@]}"` |
| Append elements | `arr+=(x y z)` |
| Delete element | `unset 'arr[n]'` |
| Delete whole array | `unset arr` |
| Associative array (declare) | `declare -A map` |
| Associative assignment | `map["key"]="value"` |
| Associative access | `${map["key"]}` |

---

## Indexed Arrays

```bash
# Assignment
arr[0]="first"
arr[1]="second"

# Bulk assignment (starts at index 0)
days=(Sun Mon Tue Wed Thu Fri Sat)

# With explicit indices (can be sparse)
sparse=([0]=a [5]=f [100]=z)

# Access
echo "${days[3]}"        # Wed
echo "${days[-1]}"       # Sat (negative index = from end, bash 4.3+)
```

---

## Iterating — Always Use `"${arr[@]}"`

```bash
arr=("hello world" "foo" "bar baz")

# CORRECT — preserves multi-word elements
for item in "${arr[@]}"; do
    echo "$item"
done
# prints:  hello world / foo / bar baz

# WRONG — word-splits, breaks spaces
for item in ${arr[@]}; do ...   # don't do this
for item in "${arr[*]}"; do ... # one big string — don't do this
```

Same rule as `"$@"`: always quote `[@]`.

---

## Array Metadata

```bash
arr=(a b c d e)

echo "${#arr[@]}"        # 5 — number of elements
echo "${#arr[2]}"        # 1 — length of element at index 2

# Sparse array — get actual indices used
sparse=([2]=a [7]=b [42]=c)
echo "${!sparse[@]}"     # 2 7 42

# Iterate over indices
for i in "${!arr[@]}"; do
    echo "[$i] = ${arr[i]}"
done
```

---

## Appending

```bash
arr=(a b c)
arr+=(d e f)
echo "${arr[@]}"         # a b c d e f

# Add single element
arr+=("new item")
```

---

## Deleting

```bash
arr=(a b c d e f)

unset 'arr[2]'           # delete index 2 — quotes prevent glob expansion
echo "${arr[@]}"         # a b d e f  (index 2 is gone, gap remains)
echo "${#arr[@]}"        # 5

unset arr                # delete entire array

# NOTE: arr= or arr[0]= does NOT clear the array — it only sets element 0
arr=(a b c)
arr=                     # only clears element 0 (arr[0])
echo "${arr[@]}"         # b c  — rest remains!
```

---

## Sorting

No built-in sort — pipe through `sort`:

```bash
arr=(f e d c b a)
sorted=($(for i in "${arr[@]}"; do echo "$i"; done | sort))
echo "${sorted[@]}"      # a b c d e f

# Sort numerically
nums=(10 2 30 4 1)
sorted=($(printf '%s\n' "${nums[@]}" | sort -n))

# Sort files by modification time
mapfile -t files < <(ls -t *.log)   # mapfile reads lines into array
```

---

## Associative Arrays (bash 4.0+)

Must be explicitly declared with `declare -A`:

```bash
declare -A colors
colors["red"]="#ff0000"
colors["green"]="#00ff00"
colors["blue"]="#0000ff"

echo "${colors["blue"]}"     # #0000ff

# Iterate values
for color in "${colors[@]}"; do
    echo "$color"
done

# Iterate keys
for name in "${!colors[@]}"; do
    echo "$name = ${colors[$name]}"
done

# Check if key exists
if [[ -v colors["red"] ]]; then
    echo "red is defined"
fi

# Count occurrences (classic use case)
declare -A count
for word in $(cat /etc/hosts); do
    (( ++count["$word"] ))
done
for key in "${!count[@]}"; do
    echo "${count[$key]} $key"
done | sort -rn | head
```

---

## `mapfile` / `readarray` — Read Lines Into Array

```bash
# Read file into array (one element per line)
mapfile -t lines < /etc/hosts
echo "${#lines[@]}"          # number of lines
echo "${lines[0]}"           # first line

# From command output
mapfile -t procs < <(ps aux | tail -n +2)

# Equivalent manual method
while IFS= read -r line; do
    arr+=("$line")
done < file.txt
```

> `mapfile` is cleaner than the while-read loop for loading whole files. Available in bash 4.0+.

---

## Practical Patterns

```bash
# Stack (push/pop)
stack=()
stack+=("item1")           # push
stack+=("item2")
last="${stack[-1]}"        # peek
unset 'stack[-1]'          # pop

# Queue (enqueue/dequeue)
queue=()
queue+=("item")            # enqueue
first="${queue[0]}"        # peek front
queue=("${queue[@]:1}")    # dequeue (slice from index 1)

# Collect matching files
declare -a logs
for f in /var/log/*.log; do
    [[ -r "$f" ]] && logs+=("$f")
done

# Parallel job tracking with PIDs
declare -a pids
for f in *.mkv; do
    ffmpeg -i "$f" "${f%.mkv}.mp4" &
    pids+=($!)
done
for pid in "${pids[@]}"; do
    wait "$pid"
done

# Docker container list
mapfile -t containers < <(docker ps --format '{{.Names}}')
for c in "${containers[@]}"; do
    echo "Container: $c"
done

# Count file owners (associative array)
declare -A owner_count
for f in /home/pango/*; do
    owner=$(stat -c %U "$f")
    (( ++owner_count["$owner"] ))
done
```

---

## Notes

- **Bash arrays are 1-dimensional** — no 2D arrays; simulate with flat index: `arr[$((i*cols + j))]`
- **Sparse arrays**: indices don't have to be contiguous; `${#arr[@]}` counts actual elements, not the range
- **zsh**: arrays are 1-indexed by default (not 0-indexed like bash); use `setopt KSH_ARRAYS` in zsh for 0-based; associative arrays use `declare -A` same as bash
- **`mapfile`** is bash-only; zsh has `arr=("${(@f)$(cat file)}")` as equivalent
