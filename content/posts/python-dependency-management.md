---
categories:
    - python
date: 2025-03-09T13:15:22Z
description: A comprehensive comparison of Python's two approaches for grouping dependencies - Dependency Groups (PEP 735) and Optional Dependencies (PEP 631) - with practical examples using uv
keywords: dependency groups, optional dependencies, PEP 735, PEP 631, python packaging, uv
tags:
    - python
    - pip
    - uv
    - dependencies
    - pep
title: 'Understanding Python Dependency Management: Groups vs Optional Dependencies'
---



# Dependency Groups

Currently, [PEP](https://peps.python.org/) supports two approaches for grouping extra dependencies:

- [PEP 735: Dependency Groups](https://peps.python.org/pep-0735/)
- [PEP 631: Optional Dependencies](https://peps.python.org/pep-0631/#optional-dependencies)

In this document, we introduce both approaches using `uv`.

## Dependency Groups

If dependencies are only required for project development, we can use `dependency-groups` to manage them. For example, development tools like `pytest`
and `ruff` are needed for development but not for production.

```toml
[dependency-groups]
cuda = [
    "torch>=2.6.0",
    "megatron-core",
]
custom = [
    "torch",
    "megatron-core",
]

[tool.uv]
conflicts = [
    [
        { group = "cuda" },
        { group = "custom" },
    ],
]

[tool.uv.sources]
megatron-core = [
    { git = "https://github.com/NVIDIA/Megatron-LM", tag = "core_r0.8.0", group = "cuda" },
    { git = "http://xxx.git", branch = "main", group = "custom" },
]
torch = { url = "http://xxx.whl", group = "custom" }
```

To install dependencies from a specific group, use:

```bash
uv sync --group cuda
```

## Optional Dependencies

If dependencies are intended for package users, we can use `optional-dependencies` to manage them. For example, `torch` offers both CPU and GPU
versions, and we can differentiate them using `optional-dependencies`.

```toml
[project.optional-dependencies]
cuda = [
    "torch>=2.6.0",
    "megatron-core",
]
custom = [
    "torch",
    "megatron-core",
]

[tool.uv]
conflicts = [
    [
        { extra = "cuda" },
        { extra = "custom" },
    ],
]

[tool.uv.sources]
megatron-core = [
    { git = "https://github.com/NVIDIA/Megatron-LM", tag = "core_r0.8.0", extra = "cuda" },
    { git = "http://xxx.git", branch = "main", extra = "custom" },
]
torch = { url = "http://xxx.whl", extra = "custom" }
```

To install optional dependencies, use:

```bash
uv sync --extra cuda

# install this package
uv add 'this-package[cuda]'
```

## Conclusion

Both approaches group dependencies, but the key difference is their target audience:

- `dependency-groups` are used for managing dependencies needed by project developers.
- `optional-dependencies` are used for managing dependencies required by package users.
