# Regexp

A small regular expression engine written in pure Python (no third-party dependencies). It parses a pattern into a syntax tree, builds an NFA and then a minimized DFA, and supports matching plus two DFA-level operations: inversion and addition of symbols. A DFA can also be converted back into a regular expression.

## Features

- Parsing of a pattern into a binary syntax tree
- Literals, alternation `|`, Kleene star `*`, optional `?`
- Exact repetition `{n}` applied to the preceding item
- Named groups `(<name>...)`
- Shielded literals `%...%`, so symbols like `*` or `|` can be matched as plain characters
- Two matching modes: tree-based (`tree_match`) and DFA-based (`dfa_passage`)
- DFA minimization (partition refinement)
- Language inversion (reversed automaton)
- Addition of new symbols to the alphabet
- Conversion of a DFA back to a regular expression (state elimination)

## Project structure

| File | Purpose |
|------|---------|
| `TreenParser.py` | Tokenizer (`Parser`), syntax tree builder (`tree_builder`), and tree matcher (`Tree_passage`) |
| `Authomat.py` | NFA and DFA classes, NFA construction, subset construction (`DFA_builder`), minimization (`min_DFA2`) |
| `Operations.py` | DFA inversion (`inversion`), alphabet extension (`addition`), and DFA-to-regex conversion (`restore`) |
| `Regex.py` | High-level `regex` class that ties everything together, plus a demo script |
| `beta.py` | Experimental scratch code (older minimization and matcher variants) |

## Usage

```python
from Regex import regex

r = regex('a(s{2}da)?a|ds*')
r.compile()                    # builds NFA, DFA and minimized DFA

r.tree_match('dss')            # set of end positions reached by the tree matcher
r.dfa_passage('dss')           # matched length via DFA, 0 if no match

inv = r.inversion()            # regex for the reversed language
add = r.addition({'x'})        # same language over an extended alphabet (None if unchanged)
print(inv.regex, add.regex)
print(r.restore())             # regex reconstructed from the DFA
```

Run the demo:

```
python Regex.py
```

## Pattern syntax

| Syntax | Meaning |
|--------|---------|
| `abc` | Literal characters |
| `a\|b` | Alternation |
| `a*` | Zero or more |
| `a?` | Zero or one |
| `a{3}` | Exactly three repetitions of `a` |
| `(...)` | Group |
| `(<name>...)` | Named group |
| `%*%` | Shielded literal (matches `*` as a character) |

## Known limitations

- `dfa_passage` returns the position where the DFA stops, not the longest accepted prefix, and returns `0` both for "no match" and "empty match".
- Unbalanced parentheses and an empty pattern are not validated and may cause `tree_builder` to loop or fail.
- `{0}` is not handled correctly by either matcher.
- `tree_match` returns a set of positions rather than a boolean.
- `Regex.py` runs its demo code at import time, outside the `if __name__ == "__main__"` guard.

These are noted for future work; see the code review for details.
