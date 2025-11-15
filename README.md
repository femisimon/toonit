# toonit
[![CI](https://github.com/femisimon/toonit/actions/workflows/ci.yml/badge.svg)](https://github.com/femisimon/toonit/actions/workflows/ci.yml)
[![PyPI Version](https://img.shields.io/pypi/v/toonit)](https://pypi.org/project/toonit/)
[![PyPI Downloads](https://img.shields.io/pypi/dm/toonit)](https://pypi.org/project/toonit/)
[![Python Versions](https://img.shields.io/pypi/pyversions/toonit)](https://pypi.org/project/toonit/)
[![License](https://img.shields.io/github/license/femisimon/toonit)](LICENSE)

[→ Visit the project on GitHub](https://github.com/femisimon/toonit)

toonit is a developer-friendly serializer for **TOON – Token-Oriented Object Notation**.  
It ships a Python scanner, parser, encoder, decoder, and CLI so you can round-trip Python
structures through the TOON format without hand-writing parsers.

## Features
- Fast encode/decode with perfect round-tripping
- Top-level key folding (`a.b: 1` style) with automatic unfolding on decode
- Columnar encoding for lists of dictionaries and analytics payloads
- Friendly CLI: `toonit encode`, `toonit decode`
- Works with any modern Python (3.9+)

---

## Installation

Published package:

```bash
pip install toonit
```

Editable install for development:

```bash
pip install -e .
```

## Why TOON?

TOON targets structured data where JSON becomes verbose. Compared to JSON it is:

- Smaller for nested or repetitive payloads (often 40–70% savings)
- Friendly for analytics because columnar expansion is built in
- Easier to read and edit thanks to lightweight syntax
- Safer for config files because comments and trailing commas are forbidden by design

## Quick Start

### Encode Python → TOON

```python
import toonit

payload = {
    "user": {"id": 1, "name": "Femi"},
    "features": ["folding", "columnar", "cli"],
}

text = toonit.encode(payload)
print(text)
```

### Decode TOON → Python

```python
raw = """
user.id: 1,
user.name: "Femi",
features: ["folding", "columnar", "cli"]
"""

data = toonit.decode(raw)
print(data["user"]["name"])
```

### Round-trip guarantee

```python
original = {"info": {"name": "Femi", "city": "Lagos"}}
assert toonit.decode(toonit.encode(original)) == original
```

### CLI streaming

```bash
curl https://example.com/data.json | toonit encode > backup.toon
toonit decode backup.toon | jq '.info.name'
```

### Writing adapters

```python
from dataclasses import asdict, dataclass
import toonit

@dataclass
class Project:
    name: str
    metrics: dict[str, int]

def serialize(project: Project) -> str:
    return toonit.encode(asdict(project))

proj = Project("Alpha", {"latency_ms": 12})
print(serialize(proj))
```

## Folding Rules

Nested dictionaries are folded into dotted keys during encoding:

```python
toonit.encode({"a": {"b": 1, "c": 2}})
```

produces TOON like:

```
a.b: 1,
a.c: 2
```

During decoding those dotted keys are unfolded back to their original hierarchy.

## Columnar Encoding

Lists of dictionaries are encoded column-by-column automatically:

```python
rows = [{"x": 1, "y": 10}, {"x": 2, "y": 20}, {"x": 3, "y": 30}]

# Great for analytics exports or tabular logs
encoded = toonit.encode(rows)
```

which yields TOON similar to:

```
x: [1, 2, 3],
y: [10, 20, 30]
```

Decoding brings the rows back intact:

```python
assert toonit.decode(encoded) == rows
```

### Mixing Columnar + Folding

```python
projects = [
    {"meta": {"name": "Alpha"}, "metrics": {"latency": 10}},
    {"meta": {"name": "Beta"}, "metrics": {"latency": 20}},
]

text = toonit.encode(projects)
print(text)
```

## CLI

```
toonit encode input.json > out.toon
toonit decode out.toon
echo '{"a": 1, "b": 2}' | toonit encode
```

## Tests

```
pytest
```

The suite covers parsing, scanning, folding, columnar transforms, error cases, and round-trip behavior.

## Project Layout

```
toonit/
  core/       # scanner, parser, AST, writer
  encoding/   # folding + columnar transforms
  decoding/   # normalization + validation
  cli.py      # command-line entry points
  api.py      # public encode/decode API
```

## Example TOON

```
a.b: 1,
list: [1, 2, 3],
info.name: "Femi",
info.city: "Lagos"
```

## License

MIT License. See `LICENSE` for details.

## Contributing

Issues and pull requests are welcome! File a ticket, suggest a feature, or open a discussion if you have ideas for the TOON format or the Python implementation.
