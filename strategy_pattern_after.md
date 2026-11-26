# How to export help articles in multiple formats with Python

This guide is for Python developers who want to add output formats without piling up `if` statements. You'll build a script that exports help articles as JSON or Markdown, using the Strategy pattern. You need Python 3.11 or later.

## Step 1. Write one function per format

Each function is a *strategy* that takes the same input and returns text. Save this code as `export_articles.py`.

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
```

**Checkpoint.** Run `python -c "from export_articles import *; print(to_json(ARTICLES[:1]))"`. You should see the first article as JSON.

## Step 2. Select a strategy by name

Now you'll add a dictionary that maps each format name to its function. Add this code to the end of the file.

```python
Exporter = Callable[[list[dict[str, str]]], str]
EXPORTERS: dict[str, Exporter] = {"json": to_json, "markdown": to_markdown}


def export(articles: list[dict[str, str]], format_name: str) -> str:
    if format_name not in EXPORTERS:
        choices = ", ".join(EXPORTERS)
        raise ValueError(f"Unknown format {format_name!r}. Choose from {choices}.")
    return EXPORTERS[format_name](articles)
```

**Checkpoint.** Run `python -c "from export_articles import *; export(ARTICLES, 'xml')"`. The last line of the error should read `ValueError: Unknown format 'xml'. Choose from json, markdown.`

## Step 3. Run the export

Add this code to the end of the file, then run `python export_articles.py`.

```python
if __name__ == "__main__":
    print(export(ARTICLES, "markdown"))
```

You should see this table source. The backslash keeps the `|` inside a title from splitting its cell.

```text
| Title | Status |
| --- | --- |
| Reset your password | published |
| Set up SSO \| SAML | draft |
```

## What you built

Your script now exports articles as JSON or Markdown through a single `export` function.

**Next step.** Add CSV by writing a `to_csv` function with Python's `csv` module and registering it in `EXPORTERS`. You won't need to change `export`.
