# Chapter 34 – Strings and Numbers

## Quick Reference — Parameter Expansions

| Expansion | Effect |
|-----------|--------|
| `${var}` | Basic expansion (use when adjacent to text) |
| `${var:-default}` | Use `default` if var unset/empty |
| `${var:=default}` | Assign and use `default` if unset/empty |
| `${var:?message}` | Exit with error if unset/empty |
| `${var:+altval}` | Use `altval` if var is set and non-empty |
| `${#var}` | String length |
| `${var:offset}` | Substring from offset |
| `${var:offset:len}` | Substring of length len from offset |
| `${var: -N}` | Last N chars (space before `-` required) |
| `${var#pattern}` | Remove shortest prefix match |
| `${var##pattern}` | Remove longest prefix match |
| `${var%pattern}` | Remove shortest suffix match |
| `${var%%pattern}` | Remove longest suffix match |
| `${var/pat/str}` | Replace first match |
| `${var//pat/str}` | Replace all matches |
| `${var/#pat/str}` | Replace if match at start |
| `${var/%pat/str}` | Replace if match at end |
| `${var,,}` | All lowercase |
| `${var^^}` | All uppercase |
| `${var,}` | First char lowercase |
| `${var^}` | First char uppercase |
| `${!prefix*}` | Variable names starting with prefix |

---

## Default Value Expansions

```bash
# Use default if unset or empty (doesn't assign)
echo "${name:-anonymous}"

# Assign default if unset or empty
: "${name:=anonymous}"      # : is a no-op command; forces expansion
echo "$name"                # now "anonymous" if it was empty

# Exit with error if unset or empty
echo "${1:?Usage: $0 <filename>}"

# Use alternate value only when var IS set
echo "${debug:+--verbose}"  # outputs --verbose only if debug is set
```

---

## String Length and Substring

```bash
str="Hello, World"

echo "${#str}"          # 12 — length

echo "${str:7}"         # World — from index 7
echo "${str:7:5}"       # World — 5 chars from index 7
echo "${str: -5}"       # World — last 5 chars (space before -)
echo "${str: -5:3}"     # Wor   — 3 chars starting from -5
```

---

## Prefix / Suffix Stripping

```bash
file="archive.tar.gz"

echo "${file#*.}"       # tar.gz  — strip shortest prefix up to first .
echo "${file##*.}"      # gz      — strip longest prefix up to last .
echo "${file%.*}"       # archive.tar — strip shortest suffix from last .
echo "${file%%.*}"      # archive     — strip longest suffix from first .

# Practical: get basename without extension
name="${file##*/}"      # strip path (like basename)
ext="${file##*.}"       # extension only
base="${file%.*}"       # filename without last extension

# Strip leading zero (needed before arithmetic — shell treats 08 as invalid octal)
hour="08"
hour="${hour#0}"        # "8"
```

---

## Search and Replace

```bash
foo="JPG.JPG.JPG"

echo "${foo/JPG/jpg}"   # jpg.JPG.JPG  — first only
echo "${foo//JPG/jpg}"  # jpg.jpg.jpg  — all
echo "${foo/#JPG/jpg}"  # jpg.JPG.JPG  — only at start
echo "${foo/%JPG/jpg}"  # JPG.JPG.jpg  — only at end
echo "${foo//JPG/}"     # ..           — delete all (omit replacement)

# Batch replace spaces with underscores
filename="my file name.txt"
echo "${filename// /_}"  # my_file_name.txt
```

---

## Case Conversion

```bash
str="Hello World"

echo "${str,,}"         # hello world   — all lowercase
echo "${str^^}"         # HELLO WORLD   — all uppercase
echo "${str,}"          # hELLO wORLD   — first char lowercase  (wait: only first char of whole string)
echo "${str^}"          # Hello World   — first char uppercase

# Normalize input before comparison
input="YES"
[[ "${input,,}" == "yes" ]] && echo "confirmed"
```

Via `declare` (enforce on assignment):
```bash
declare -l lower_var    # always stored lowercase
declare -u upper_var    # always stored uppercase
lower_var="HeLLo"       # stored as "hello"
```

---

## Arithmetic `$(( ))`

```bash
echo $(( 2 + 3 ))       # 5
echo $(( 10 / 3 ))      # 3  (integer division)
echo $(( 10 % 3 ))      # 1  (modulo / remainder)
echo $(( 2 ** 8 ))      # 256 (exponentiation)
```

### Number bases

```bash
echo $(( 0xff ))        # 255  — hex
echo $(( 0777 ))        # 511  — octal (leading zero)
echo $(( 2#11111111 ))  # 255  — binary (base#number)
echo $(( 16#ff ))       # 255  — explicit hex
```

### Assignment operators

```bash
(( x = 5 ))             # assign
(( x += 3 ))            # x = x + 3
(( x -= 1 ))            # x = x - 1
(( x *= 2 ))
(( x /= 4 ))
(( x %= 3 ))
(( x++ ))               # post-increment (returns old value)
(( ++x ))               # pre-increment (returns new value) — prefer this
(( x-- ))
(( --x ))
```

### Bit operations

```bash
echo $(( 1 << 4 ))      # 16   — left shift (multiply by 2^4)
echo $(( 128 >> 2 ))    # 32   — right shift (divide by 4)
echo $(( 0xff & 0x0f )) # 15   — bitwise AND
echo $(( 0xf0 | 0x0f )) # 255  — bitwise OR
echo $(( 0xff ^ 0x0f )) # 240  — bitwise XOR

# Powers of 2
for (( i=0; i<8; i++ )); do echo $(( 1<<i )); done
```

### Ternary operator

```bash
# condition ? true_expr : false_expr
a=0
(( a < 1 ? (a+=1) : (a-=1) ))  # toggle between 0 and 1

# Inline min/max
(( max = a > b ? a : b ))
```

---

## `bc` — Floating-Point and Higher Math

Shell arithmetic is integers only. Use `bc` for floats.

```bash
# Here string (quickest)
bc <<< "scale=4; 22/7"          # 3.1428
bc <<< "scale=2; sqrt(2)"       # 1.41

# Pipe
echo "scale=10; 1/3" | bc       # 0.3333333333

# Here document for multi-line
result=$(bc <<- EOF
    scale = 6
    rate = 0.0775 / 12
    principal = 135000
    months = 180
    principal * ((rate * (1 + rate)^months) / ((1 + rate)^months - 1))
EOF
)
echo "$result"

# bc -l loads math library (sin, cos, sqrt, etc.)
bc -l <<< "s(3.14159/2)"        # sin(π/2) ≈ 1.0
```

`scale` sets decimal places. Default is 0 (integer results even in bc).

---

## Efficiency: Parameter Expansion vs Subshells

```bash
# Slow: spawns subshell for each iteration
len="$(echo -n "$word" | wc -c)"

# Fast: pure parameter expansion, no subshell
len="${#word}"
```

The book demonstrated 3.6s vs 0.06s — a 60x speedup. Prefer `${#var}` over `wc -c` in loops.

---

## Practical Patterns

```bash
# Safe basename (no external process)
PROGNAME="${0##*/}"

# Extension detection
ext="${filename##*.}"
case "$ext" in
    gz|bz2|xz|zst) echo "compressed" ;;
    *) echo "other" ;;
esac

# Pad number with leading zeros
printf "%05d\n" "$n"            # printf is better for formatting
# Or:
n=7; echo "${n}" → need printf for zero-padding, not parameter expansion

# Default config path
config="${XDG_CONFIG_HOME:-$HOME/.config}/myapp/config"

# Require argument or die
input="${1:?Error: provide a filename}"

# Toggle flag
flag="${flag:+}" # clear flag if set... actually use arithmetic:
(( flag ^= 1 ))  # XOR toggle between 0 and 1

# Strip ANSI color codes
clean="${str//$'\e['[0-9]*m/}"   # approximate; use sed for robustness
```
