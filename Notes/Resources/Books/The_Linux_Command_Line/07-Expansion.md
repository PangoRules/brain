### Expansion types (processed before command runs)

| Type | Syntax | Example |
|------|--------|---------|
| Pathname | wildcards | `echo D*` |
| Tilde | `~` | `echo ~` -> `/home/me` |
| Arithmetic | `$((expr))` | `echo $((2+2))` -> `4` |
| Brace | `{a,b,c}` | `echo {A,B,C}` -> `A B C` |
| Parameter | `$VAR` | `echo $USER` |
| Command sub | `$(cmd)` | `echo $(whoami)` |

### Arithmetic operators
`+` `-` `*` `/` `%` (modulo) `**` (exponent) -- integers only

### Brace expansion examples
```bash
echo {01..12}              # 01 02 03 ... 12
echo {Z..A}                # reverse alphabet
mkdir {2024..2026}-{01..12} # 36 directories
echo a{A{1,2},B{3,4}}b    # aA1b aA2b aB3b aB4b
```

### Quoting

| Type | Suppresses | Allows |
|------|-----------|--------|
| `"double"` | Word splitting, pathname/brace/tilde expansion | `$var`, `$(cmd)`, `$((...))` |
| `'single'` | **Everything** | Nothing |
| `\` (escape) | Next character's special meaning | -- |

```bash
echo "The cost is \$5.00"   # escape the $
echo "$(cal)"               # preserves newlines
mv bad\&file good_file      # escape special chars
```
