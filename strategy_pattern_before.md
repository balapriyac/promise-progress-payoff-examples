# The Strategy Pattern in Python

This document covers the Strategy pattern.

## Background

Strategy is a behavioral design pattern. It defines a family of interchangeable algorithms and lets the calling code choose one at runtime. Classic descriptions use an interface with several concrete classes. Python functions are first-class objects, so a function can fill the role of a strategy.

## Design considerations

A dictionary can map names to functions, which avoids a growing chain of conditionals. The functions should share a signature. The code should also decide how to treat an unknown name. For very small programs with a single behavior, the pattern may add more structure than needed.

## Implementation

```python
import json
from collections.abc import Callable

ARTICLES = [
    {"title": "Reset your password", "status": "published"},
    {"title": "Set up SSO | SAML", "status": "draft"},
]


def to_json(articles: list[dict[str, str]]) -> str:
    return json.dumps(articles, indent=2)


def to_markdown(articles: list[dict[str, str]]) -> str:
    rows = ["| Title | Status |", "| --- | --- |"]
    for article in articles:
        title = article["title"].replace("|", r"\|")
        rows.append(f"| {title} | {article['status']} |")
    return "\n".join(rows)


Exporter = Callable[[list[dict[str, str]]], str]
EXPORTERS: dict[str, Exporter] = {"json": to_json, "markdown": to_markdown}


def export(articles: list[dict[str, str]], format_name: str) -> str:
    if format_name not in EXPORTERS:
        choices = ", ".join(EXPORTERS)
        raise ValueError(f"Unknown format {format_name!r}. Choose from {choices}.")
    return EXPORTERS[format_name](articles)


if __name__ == "__main__":
    print(export(ARTICLES, "markdown"))
```
