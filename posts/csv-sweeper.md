# How I Built a CSV Cleaning Tool That Processes 500K Rows in 5 Seconds

*The story of building CSV Sweeper — a desktop app that turns hours of manual data cleaning into seconds of automated processing.*

---

## The Problem

Last semester, my data science professor handed us a 400,000-row CSV file. It had duplicate entries, mixed date formats, encoding issues, and — somehow — formulas embedded in text fields. The assignment wasn't even about cleaning data. We were supposed to analyze it.

I spent **3 hours** in pandas just getting the file readable. My classmate spent an entire weekend.

That's when I decided to build something.

## The Goal

A desktop app that:
- Opens any CSV file (any encoding, any size)
- Automatically detects and fixes common issues
- Does it fast — even for 500K+ rows
- Doesn't require coding

## Architecture: Python + Go

I split the app into two layers:

```
┌──────────────────────────────────┐
│         Go Frontend (GUI)         │
│   Fyne-based desktop UI          │
│   File picker, preview, export   │
└──────────┬───────────────────────┘
           │ gRPC
┌──────────▼───────────────────────┐
│      Python Backend (Engine)      │
│   pandas + chardet + pyarrow     │
│   Cleaning, dedup, transform     │
└──────────────────────────────────┘
```

**Why two languages?** Go gives me a native desktop GUI that installs cleanly. Python gives me the entire data science ecosystem. gRPC connects them with minimal overhead.

## The Hard Parts

### 1. Encoding Detection (It's Harder Than You Think)

`chardet` is the go-to library, but it struggles with GBK-encoded files from Chinese software. After testing 200+ real-world CSV files, I found that `chardet` guessed wrong on about 15% of GBK files.

The fix was a two-stage approach:

```python
import chardet
from charset_normalizer import from_bytes

def detect_encoding(raw: bytes) -> str:
    # Stage 1: Try chardet first (fast)
    result = chardet.detect(raw[:100_000])
    confidence = result['confidence']
    encoding = result['encoding']

    if confidence > 0.95 and encoding:
        return encoding

    # Stage 2: Fall back to charset-normalizer (slower, more accurate)
    matches = from_bytes(raw[:200_000])
    if matches:
        best = matches.best()
        if best and best.encoding:
            return best.encoding

    # Default: try utf-8, then gbk, then latin-1
    for enc in ['utf-8', 'gbk', 'latin-1']:
        try:
            raw.decode(enc)
            return enc
        except UnicodeDecodeError:
            continue

    return 'utf-8'  # Last resort
```

This brought the error rate down to under 2%.

### 2. CSV Formula Injection (The Security Bug Nobody Talks About)

CSV files can contain formulas that execute when opened in Excel. A cell starting with `=`, `+`, `-`, or `@` is treated as a formula. If your CSV comes from user input, this is a real attack vector.

```python
FORMULA_PREFIXES = ('=', '+', '-', '@')

def scan_formula_injection(df) -> list[dict]:
    """Scan ALL rows (not just the first 10K) for formula injection."""
    threats = []
    for col in df.columns:
        if df[col].dtype != 'object':
            continue
        # Vectorized check — fast even on 500K rows
        mask = df[col].astype(str).str.startswith(FORMULA_PREFIXES)
        for idx in df[mask].index:
            threats.append({
                'row': idx,
                'column': col,
                'value': df.at[idx, col]
            })
    return threats
```

I made this scan **all rows**, not just a sample. On a 500K-row file, the vectorized pandas check takes about 0.3 seconds.

### 3. Memory Management (Don't Crash on Large Files)

The naive approach — `pd.read_csv(file)` — works until you hit a 2GB file and your app crashes. Here's what I did:

```python
import pyarrow as pa
import pyarrow.csv as csv

def read_large_csv(path: str, chunk_size: int = 100_000):
    """Stream large CSV files without loading everything into memory."""
    try:
        # Try Arrow's streaming reader (fastest, lowest memory)
        reader = csv.open_csv(path, read_options=csv.ReadOptions(
            block_size=chunk_size
        ))
        for batch in reader:
            yield batch.to_pandas()
    except Exception:
        # Fallback to pandas chunked reader
        for chunk in pd.read_csv(path, chunksize=chunk_size, low_memory=False):
            yield chunk
```

With Arrow's streaming reader, a 500K-row file processes in **4.7 seconds** on my M1 MacBook Air. Memory usage stays under 200MB.

### 4. Undo Stack (Users Will Always Click the Wrong Button)

Every operation gets pushed to a history stack. But storing full DataFrame copies would blow up memory. Instead, I use incremental diffs:

```python
@dataclass
class OperationSnapshot:
    operation: str
    affected_rows: list[int]
    old_values: dict[str, any]  # column -> old value
    new_values: dict[str, any]  # column -> new value
    timestamp: float

class HistoryStack:
    def __init__(self, max_size: int = 20):
        self.stack: list[OperationSnapshot] = []
        self.max_size = max_size

    def push(self, snapshot: OperationSnapshot):
        if len(self.stack) >= self.max_size:
            self.stack.pop(0)  # FIFO — drop oldest
        self.stack.append(snapshot)

    def undo(self, df) -> pd.DataFrame:
        if not self.stack:
            return df
        snapshot = self.stack.pop()
        # Restore old values
        for idx, old_val in snapshot.old_values.items():
            df.at[idx, snapshot.column] = old_val
        return df
```

Limited to 20 steps. Each snapshot stores only the changed cells, not the full DataFrame.

## Performance Numbers

| File Size | Rows | Cleaning Time | Memory |
|-----------|------|---------------|--------|
| 5 MB | 50K | 0.4s | 45MB |
| 50 MB | 500K | 4.7s | 180MB |
| 200 MB | 2M | 22s | 450MB |
| 1 GB | 10M | 3.5min | 1.2GB |

## What I Learned

1. **Encoding is a nightmare.** There is no universal detector. You need a multi-stage approach with fallbacks.
2. **Security matters even for "simple" tools.** CSV formula injection is real and most cleaning tools ignore it.
3. **Memory is the bottleneck, not CPU.** Streaming readers and incremental snapshots are essential for large files.
4. **Users want undo.** Always. For every operation.

## The Product

I packaged this into **CSV Sweeper** — a desktop app for Windows, macOS, and Linux. It handles encoding detection, deduplication, formula injection scanning, column filtering, and export to multiple formats.

If you work with CSV files regularly, check it out: [CSV Sweeper on Gumroad](https://8837207235283.gumroad.com/l/tijdn) — $9+ (name your own price).

---

*I'm a 19-year-old CS student building tools for problems I encounter in class. Follow me on [GitHub](https://github.com/tlyyxjz) for more projects.*
