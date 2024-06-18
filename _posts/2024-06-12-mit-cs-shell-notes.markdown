---
layout: post
title:  "2024-06-12 MIT CS shell"
date:   2024-06-12 1:53:49 -0500
categories: algorithm
---

#### Regax of sed
- `.`: means “any single character” except newline
- `*`: zero or more of the preceding match
- `+`: one or more of the preceding match
- `[abc]`: any one character of a, b, and c
- `(RX1|RX2)`: either something that matches RX1 or RX2
- `^`: the start of the line
- `$`: the end of the line

#### GDB split screen
- `layout asm`
- `layout reg`

![gdb](/assets/2024-06-12/gdb.png)

- gdb program exec:
  + single step：`step` or `s`
  + next breaking point: `next` or `n`
  + run all program until hit the breaking point: `run` or `r`



#### How to edit a binary file by vim
- command `%!xxd` allows you to view files by vim in hex format. 
- you can also edit files in hex format by vim. 
- once you finish editing the file, you can convert the file you edited by `%!xxd -r`, and then with w command to save the file
- the check how to use xxd (hex view of a file by vim), you can use the command `:h xxd`

#### How to check the machine code of a file:
- `hexdump <exe>`

#### Several usage of git
- Create Git Patch for Specific Commit:
  + `git format-patch -1 <commit_sha>`
  + and then apply the git patch:
  + `git apply <patch file name>`

- Change previous commit order/merge commits: 
  + `git rebase -i <commit_sha> <branch name>`

- Silently commit to the current commit sha:
  + `git commit --amend --no-edit`
