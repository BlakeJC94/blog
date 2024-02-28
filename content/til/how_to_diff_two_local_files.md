+++
title = 'How_to_diff_two_local_files'
date = 2024-02-29T10:20:11+11:00
draft = true
+++

Use `git diff` with the `--no-index` flag:

```bash
git diff [<options>] --no-index [--] <path> <path>
```

Previously I've found this useful for when comparing ML configuration files where one is a modified
copy of another.

One of the comments in the Stack Overflow answer mentions the `--word-diff` flag as well, which
prints more detailed diffs.

Reference:
* [Stack Overflow](https://stackoverflow.com/a/54656539)
