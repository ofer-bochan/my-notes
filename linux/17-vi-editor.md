# Vi/Vim Editor Basics

> **What it is:** Vi (and Vim - Vi Improved) is the standard text editor on Unix/Linux systems. It's available on virtually every system, making it essential to know.

## Vi Modes

```
┌─────────────────────────────────────────────────────────────────┐
│                        Vi Modes                                  │
│                                                                  │
│   NORMAL MODE ──── i, a, o ────► INSERT MODE                    │
│        ▲                              │                          │
│        └────────── Esc ───────────────┘                          │
│        │                                                         │
│        : (colon)                                                 │
│        ▼                                                         │
│   COMMAND LINE MODE  (:w, :q, :wq, etc.)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Opening and Closing

```bash
vi filename                    # Open file
vi +10 filename                # Open at line 10
vi +/pattern filename          # Open at pattern
```

```
:w              Save
:q              Quit
:q!             Quit without saving
:wq             Save and quit
ZZ              Save and quit (no colon)
```

## Entering Insert Mode

```
i       Insert before cursor
I       Insert at beginning of line
a       Append after cursor
A       Append at end of line
o       Open new line below
O       Open new line above
```

## Navigation

```
h       Left
j       Down
k       Up
l       Right
w       Next word
b       Previous word
0       Beginning of line
$       End of line
gg      First line
G       Last line
:10     Go to line 10
Ctrl+f  Page down
Ctrl+b  Page up
```

## Editing

```
x       Delete character
dw      Delete word
dd      Delete line
d$      Delete to end of line
D       Delete to end of line

yy      Yank (copy) line
yw      Yank word
p       Paste after cursor
P       Paste before cursor

u       Undo
Ctrl+r  Redo
.       Repeat last command
```

## Search and Replace

```
/pattern        Search forward
?pattern        Search backward
n               Next match
N               Previous match

:s/old/new/     Replace first on line
:s/old/new/g    Replace all on line
:%s/old/new/g   Replace all in file
:%s/old/new/gc  Replace with confirmation
```

## Visual Mode

```
v       Character selection
V       Line selection
Ctrl+v  Block selection

# After selecting:
d       Delete
y       Yank (copy)
>       Indent
<       Unindent
```

## Useful Settings

```
:set number          Show line numbers
:set nonumber        Hide line numbers
:set ignorecase      Case insensitive search
:set hlsearch        Highlight search
:noh                 Clear highlight
:set paste           Paste mode (no auto-indent)
:set tabstop=4       Tab width
:set expandtab       Spaces instead of tabs
:syntax on           Syntax highlighting
```

## Quick Reference Card

```
ESSENTIAL
─────────
i       Insert mode
Esc     Normal mode
:w      Save
:q      Quit
:wq     Save and quit

MOVEMENT
─────────
h/j/k/l Arrow keys
gg      Start of file
G       End of file
:123    Go to line

EDITING
─────────
dd      Delete line
yy      Copy line
p       Paste
u       Undo
/text   Search
```

---

*Part of the [Linux Documentation](README.md)*

