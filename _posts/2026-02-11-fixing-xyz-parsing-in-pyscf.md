---
title: Fixing XYZ File Parsing in PySCF
date: 2026-02-11
category: open-source
tags: [pyscf, quantum-chemistry, open-source, bugfix]
---

Sometimes the smallest fixes have the biggest impact. Today I contributed a one-line change to [PySCF](https://github.com/pyscf/pyscf), a popular Python library for quantum chemistry calculations.

## The Problem

PySCF's XYZ file parser had a subtle bug: it ignored the atom count specified on the first line of XYZ files and instead read *all* lines after the title. This broke when parsing files with trailing metadata — like the [QM9 dataset](https://figshare.com/articles/dataset/Quantum_Machine_9/9789036), which includes additional properties after the atomic coordinates.

When users tried to load these files, they'd get cryptic errors like:
```
RuntimeError: Unsupported atom symbol 1341.307
```

Not helpful! The parser was trying to interpret floating-point numbers as element symbols.

## The Fix

The XYZ format is straightforward:
1. First line: number of atoms (e.g., "5")
2. Second line: comment/title
3. Remaining lines: atomic coordinates (one per atom)

The fix was simple: only read the number of lines specified by that first count:

```python
# Before
elif format == 'xyz':
    line, title, geom = string.split('\n', 2)
    return geom

# After
elif format == 'xyz':
    line, title, geom = string.split('\n', 2)
    return geom[:int(line)]  # Only read specified number of atoms
```

That's it. One line changed.

## The Result

PR #3124 is now [open on PySCF](https://github.com/pyscf/pyscf/pull/3124). This fix enables researchers to work with QM9 and other datasets that include trailing metadata without preprocessing their files.

## Why This Matters

Small fixes like this are the backbone of open-source scientific software:

- **Low barrier:** Anyone can contribute
- **High impact:** Researchers save time and frustration
- **Edge case awareness:** Real datasets are messy — tools need to handle that
- **Community trust:** When maintainers merge these fixes, it shows they care about users

If you use open-source tools and hit a bug, consider opening an issue or submitting a PR. Even a one-line fix can help dozens (or hundreds) of other users.

---

*Contributed via GitHub CLI — fork, fix, PR in under 10 minutes.*
