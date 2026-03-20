# Appendix B – Keyboard Shortcuts Reference

A comprehensive reference for terminal, Bash readline, Vim, tmux, and common application shortcuts.

---

## Table of Contents
1. [Bash / Readline Shortcuts](#1-bash--readline-shortcuts)
2. [Terminal Emulator Shortcuts](#2-terminal-emulator-shortcuts)
3. [Vim Shortcuts](#3-vim-shortcuts)
4. [Nano Shortcuts](#4-nano-shortcuts)
5. [tmux Shortcuts](#5-tmux-shortcuts)
6. [Less / Man Page Navigation](#6-less--man-page-navigation)
7. [Screen Shortcuts](#7-screen-shortcuts)

---

## 1. Bash / Readline Shortcuts

These shortcuts work in any `readline`-enabled application (bash, python REPL, gdb, etc.).
The default binding mode is **Emacs mode**. Enable Vi mode with `set -o vi`.

### Cursor Movement

| Shortcut | Action |
|----------|--------|
| `Ctrl + A` | Move cursor to **beginning** of line |
| `Ctrl + E` | Move cursor to **end** of line |
| `Ctrl + F` | Move cursor **forward** one character |
| `Ctrl + B` | Move cursor **backward** one character |
| `Alt + F` | Move cursor **forward** one word |
| `Alt + B` | Move cursor **backward** one word |
| `Ctrl + XX` | Toggle between current position and start of line |

### Editing

| Shortcut | Action |
|----------|--------|
| `Ctrl + D` | Delete character **under** cursor (or exit if line empty) |
| `Ctrl + H` | Delete character **before** cursor (same as Backspace) |
| `Ctrl + K` | Cut text from cursor to **end of line** (kill) |
| `Ctrl + U` | Cut text from cursor to **beginning of line** |
| `Ctrl + W` | Cut **previous word** |
| `Alt + D` | Cut **next word** |
| `Ctrl + Y` | Paste (yank) last cut text |
| `Alt + Y` | Cycle through kill ring after `Ctrl+Y` |
| `Ctrl + T` | **Transpose** current and previous character |
| `Alt + T` | **Transpose** current and previous word |
| `Alt + U` | Uppercase from cursor to end of word |
| `Alt + L` | Lowercase from cursor to end of word |
| `Alt + C` | Capitalize current word |
| `Ctrl + _` | **Undo** last editing action |

### History Navigation

| Shortcut | Action |
|----------|--------|
| `Ctrl + P` | Previous history entry (same as `↑`) |
| `Ctrl + N` | Next history entry (same as `↓`) |
| `Ctrl + R` | **Reverse incremental search** through history |
| `Ctrl + S` | Forward incremental history search (may need `stty -ixon`) |
| `Ctrl + G` | Abort history search, restore original line |
| `Alt + .` | Insert last argument of previous command |
| `!!` | Repeat last command |
| `!n` | Repeat command number `n` |
| `!string` | Repeat last command starting with `string` |
| `!$` | Last argument of previous command |
| `!^` | First argument of previous command |
| `!*` | All arguments of previous command |
| `^old^new` | Replace `old` with `new` in last command and run it |

### Process Control

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | Send **SIGINT** – interrupt current process |
| `Ctrl + Z` | Send **SIGTSTP** – suspend current process (use `fg`/`bg`) |
| `Ctrl + \` | Send **SIGQUIT** – quit with core dump |
| `Ctrl + L` | Clear screen (like `clear`) |
| `Ctrl + S` | **Pause** terminal output (XOFF) |
| `Ctrl + Q` | **Resume** terminal output (XON) |

### Completion

| Shortcut | Action |
|----------|--------|
| `Tab` | Auto-complete command, file, or variable |
| `Tab Tab` | Show all possible completions |
| `Alt + ?` | Show all possible completions |
| `Alt + *` | Insert all possible completions |
| `Alt + /` | Complete filename |
| `Alt + ~` | Complete username |
| `Alt + @` | Complete hostname |
| `Alt + !` | Complete command name |
| `Alt + $` | Complete variable name |

---

## 2. Terminal Emulator Shortcuts

### GNOME Terminal / Tilix

| Shortcut | Action |
|----------|--------|
| `Ctrl + Alt + T` | Open new terminal window |
| `Ctrl + Shift + T` | New **tab** |
| `Ctrl + Shift + N` | New **window** |
| `Ctrl + Shift + W` | Close tab |
| `Ctrl + Shift + Q` | Close window |
| `Ctrl + Tab` | Next tab |
| `Ctrl + Shift + Tab` | Previous tab |
| `Alt + 1–9` | Switch to tab number |
| `Ctrl + Shift + C` | Copy selection |
| `Ctrl + Shift + V` | Paste |
| `Ctrl + Shift + F` | Find in terminal |
| `Ctrl + +` | Zoom in |
| `Ctrl + -` | Zoom out |
| `Ctrl + 0` | Reset zoom |
| `F11` | Toggle fullscreen |
| `Ctrl + Shift + E` | Split pane horizontally (Tilix) |
| `Ctrl + Shift + O` | Split pane vertically (Tilix) |

### xterm / rxvt-unicode (urxvt)

| Shortcut | Action |
|----------|--------|
| `Shift + PgUp` | Scroll up |
| `Shift + PgDn` | Scroll down |
| `Ctrl + Shift + C` | Copy |
| `Ctrl + Shift + V` | Paste |

---

## 3. Vim Shortcuts

### Modes

```
Normal mode  → default; navigate and issue commands
Insert mode  → i, I, a, A, o, O to enter; Esc to return
Visual mode  → v (char), V (line), Ctrl+V (block)
Command mode → : to enter
```

### Normal Mode – Navigation

| Key | Action |
|-----|--------|
| `h j k l` | Left, down, up, right |
| `w` / `W` | Next word / WORD start |
| `b` / `B` | Previous word / WORD start |
| `e` / `E` | End of word / WORD |
| `0` | Start of line |
| `^` | First non-blank character |
| `$` | End of line |
| `gg` | Go to first line |
| `G` | Go to last line |
| `nG` or `:n` | Go to line `n` |
| `Ctrl + F` | Page down |
| `Ctrl + B` | Page up |
| `Ctrl + D` | Half-page down |
| `Ctrl + U` | Half-page up |
| `{` / `}` | Previous / next empty line (paragraph) |
| `%` | Jump to matching bracket |
| `*` | Search word under cursor (forward) |
| `#` | Search word under cursor (backward) |

### Normal Mode – Editing

| Key | Action |
|-----|--------|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dd` | Delete (cut) current line |
| `D` | Delete to end of line |
| `yy` | Yank (copy) current line |
| `p` / `P` | Paste after / before cursor |
| `u` | Undo |
| `Ctrl + R` | Redo |
| `r` | Replace single character |
| `R` | Replace mode (overwrite) |
| `c` | Change operator (e.g., `cw` = change word) |
| `C` | Change to end of line |
| `cc` | Change entire line |
| `~` | Toggle case |
| `>>` / `<<` | Indent / un-indent line |
| `.` | Repeat last change |
| `J` | Join line below to current |

### Insert Mode

| Key | Action |
|-----|--------|
| `i` | Insert before cursor |
| `I` | Insert at start of line |
| `a` | Append after cursor |
| `A` | Append at end of line |
| `o` | Open new line below |
| `O` | Open new line above |
| `s` | Substitute character (delete + insert) |
| `S` | Substitute line |
| `Ctrl + W` | Delete previous word in insert mode |
| `Ctrl + U` | Delete to start of line in insert mode |
| `Ctrl + T` | Indent current line |
| `Ctrl + D` | Un-indent current line |
| `Esc` | Return to Normal mode |

### Command Mode (`:`)

| Command | Action |
|---------|--------|
| `:w` | Save |
| `:q` | Quit |
| `:wq` or `:x` | Save and quit |
| `:q!` | Force quit without saving |
| `:e filename` | Open file |
| `:bn` / `:bp` | Next / previous buffer |
| `:ls` | List open buffers |
| `:sp` / `:vsp` | Horizontal / vertical split |
| `:%s/old/new/g` | Global find-and-replace |
| `:%s/old/new/gc` | Global replace with confirmation |
| `:n,ms/old/new/g` | Replace in line range n–m |
| `:set nu` | Show line numbers |
| `:set nonu` | Hide line numbers |
| `:set paste` | Paste mode (disable auto-indent) |
| `:syntax on` | Enable syntax highlighting |
| `:!cmd` | Run shell command |
| `:r !cmd` | Insert command output at cursor |

### Visual Mode

| Key | Action |
|-----|--------|
| `v` | Character-wise visual |
| `V` | Line-wise visual |
| `Ctrl + V` | Block visual |
| `d` | Delete selection |
| `y` | Yank selection |
| `>` / `<` | Indent / un-indent |
| `=` | Auto-indent selection |
| `:` | Start command on selection |

---

## 4. Nano Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + G` | Display help |
| `Ctrl + X` | Exit (prompts to save) |
| `Ctrl + O` | Write (save) file |
| `Ctrl + R` | Read (insert) file |
| `Ctrl + W` | Search |
| `Ctrl + \` | Search and replace |
| `Ctrl + K` | Cut current line to cut buffer |
| `Ctrl + U` | Paste from cut buffer |
| `Ctrl + ^` | Mark text (start selection) |
| `Ctrl + G` | Help |
| `Ctrl + C` | Show cursor position |
| `Ctrl + _` | Go to line number |
| `Alt + A` | Start/stop marking text |
| `Alt + U` | Undo |
| `Alt + E` | Redo |
| `Alt + #` | Toggle line numbers |
| `Alt + V` | Toggle soft-wrapping |
| `Ctrl + F` / `Ctrl + B` | Move forward / backward one character |
| `Ctrl + Space` / `Alt + Space` | Move forward / backward one word |

---

## 5. tmux Shortcuts

Default prefix: `Ctrl + B` (shown as `^B` below).

### Sessions

| Shortcut | Action |
|----------|--------|
| `tmux new -s name` | Create named session |
| `tmux ls` | List sessions |
| `tmux attach -t name` | Attach to session |
| `^B d` | **Detach** from current session |
| `^B $` | Rename current session |
| `^B s` | Switch between sessions (interactive) |
| `^B (` / `^B )` | Switch to previous / next session |
| `^B :kill-session` | Kill current session |

### Windows (Tabs)

| Shortcut | Action |
|----------|--------|
| `^B c` | Create new window |
| `^B ,` | Rename current window |
| `^B w` | List windows (interactive) |
| `^B n` / `^B p` | Next / previous window |
| `^B 0–9` | Switch to window by number |
| `^B &` | Kill current window |
| `^B l` | Last (previously used) window |

### Panes (Splits)

| Shortcut | Action |
|----------|--------|
| `^B %` | Split pane **vertically** |
| `^B "` | Split pane **horizontally** |
| `^B ←↑→↓` | Navigate panes |
| `^B o` | Cycle to next pane |
| `^B ;` | Last active pane |
| `^B q` | Show pane numbers |
| `^B z` | **Zoom** / unzoom current pane |
| `^B {` / `^B }` | Swap pane left / right |
| `^B Ctrl+←→` | Resize pane horizontally |
| `^B Ctrl+↑↓` | Resize pane vertically |
| `^B x` | Kill current pane |
| `^B !` | Break pane out to new window |
| `^B Space` | Cycle through pane layouts |

### Copy Mode

| Shortcut | Action |
|----------|--------|
| `^B [` | Enter copy mode |
| `q` or `Esc` | Exit copy mode |
| `Space` | Start selection (Vi mode) |
| `Enter` | Copy selection |
| `^B ]` | Paste buffer |
| `/` | Search forward (Vi mode) |
| `?` | Search backward (Vi mode) |
| `n` / `N` | Next / previous search match |

### Miscellaneous

| Shortcut | Action |
|----------|--------|
| `^B ?` | List all key bindings |
| `^B :` | Enter command prompt |
| `^B t` | Show clock in current pane |
| `^B ~` | Show tmux messages |
| `^B r` | Reload config (if bound) |

---

## 6. Less / Man Page Navigation

| Key | Action |
|-----|--------|
| `Space` / `f` / `Ctrl + F` | Forward one page |
| `b` / `Ctrl + B` | Backward one page |
| `d` / `Ctrl + D` | Forward half-page |
| `u` / `Ctrl + U` | Backward half-page |
| `j` / `↓` | Forward one line |
| `k` / `↑` | Backward one line |
| `g` | Go to first line |
| `G` | Go to last line |
| `ng` | Go to line `n` |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |
| `&pattern` | Show only matching lines |
| `m` + letter | Set mark |
| `'` + letter | Go to mark |
| `-N` | Toggle line numbers |
| `F` | Follow mode (like `tail -f`) |
| `s filename` | Save to file |
| `q` | Quit |
| `h` | Help |

---

## 7. Screen Shortcuts

Default escape: `Ctrl + A` (shown as `^A`).

| Shortcut | Action |
|----------|--------|
| `^A c` | Create new window |
| `^A n` / `^A p` | Next / previous window |
| `^A 0–9` | Switch to window number |
| `^A "` | List windows |
| `^A A` | Rename current window |
| `^A d` | Detach session |
| `^A k` | Kill current window |
| `^A S` | Split horizontally |
| `^A |` | Split vertically |
| `^A Tab` | Switch focus between regions |
| `^A Q` | Remove all regions except current |
| `^A [` | Enter copy/scrollback mode |
| `^A ]` | Paste |
| `^A ?` | Show key bindings |
| `^A :` | Command prompt |
| `screen -ls` | List screen sessions |
| `screen -r name` | Re-attach to session |
| `screen -S name` | Create named session |

---

*Tip: Run `bind -P` in bash to see all current readline key bindings.*
