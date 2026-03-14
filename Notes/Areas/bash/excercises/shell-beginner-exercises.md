# 🥋 Zsh Beginner Shell Exercises

## 1. Greeting Script
Ask for name and favorite language. Print a formatted message.

**Hints:**
- `read`
- `echo`
- `date`
- `$USER`

---

## 2. Positive / Negative / Zero
Validate numeric input and determine sign.

**Hints:**
- `[[ "$var" =~ regex ]]`
- `(( ))`
- `-eq`, `-lt`, `-gt`

---

## 3. Even or Odd
Check if number is even or odd.

**Hints:**
- `$(( num % 2 ))`
- `(( ))`

---

## 4. Single Word Validator
Reject empty input or multiple words.

**Hints:**
- `${=var}`
- `${#array}`
- `[[ -z "$var" ]]`

---

## 5. File Exists?
Check if argument exists and what type it is.

**Hints:**
- `-e`
- `-f`
- `-d`
- `-L`

---

## 6. Mini wc
Print line, word, character count.

**Hints:**
- `wc`
- command substitution `$( )`

---

## 7. Loop Through .txt Files
Print filename and size.

**Hints:**
- `for file in *.txt`
- `stat`
- `ls -lh`

---

## 8. Guessing Game
Generate random number and loop until correct.

**Hints:**
- `$RANDOM`
- `while`
- `read`

---

## 9. Simple Backup
Compress directory with date.

**Hints:**
- `tar -czf`
- `date +%F`
- `$1`

---

## 10. Menu Script
Create menu with 3 options using `case`.

**Hints:**
- `case`
- `while true`
- `read`