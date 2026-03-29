# Parsing Metadata from PBIX Files (Handling Multiple Internal Formats)

## Problem
Power BI `.pbix` files are not directly readable and have inconsistent internal structures. Earlier, metadata extraction was performed using PBIP files, which expose clean and accessible JSON. However, `.pbix` files are compressed binaries and do not follow a single standard format, making direct parsing significantly more complex.

---

## Approach
To address this, I designed a parser that first detects the internal structure of the `.pbix` file:

- **Old Format** → `Report/Layout`  
- **New Format** → `Report/definition/...` (PBIP-like structure)

Based on the detected format, different parsing strategies are applied to extract:

- Pages and visuals  
- Filters (report-level, page-level, and visual-level)  
- Fields and query references  

### Table Extraction Strategy
To ensure robustness, multiple fallback mechanisms were implemented:

1. **pbixray** → Extract structured metadata directly  
2. **Schema Parsing** → Parse embedded files (`DataModelSchema`, `model.bim`)  
3. **Inference** → Derive tables from query references when other methods fail  

---

## Challenges

- Lack of official documentation for `.pbix` internals  
- Co-existence of multiple internal formats  
- Encoding inconsistencies (UTF-8, UTF-16)  
- Filtering out hidden or static visuals that do not contribute meaningful insights  

---

## Outcome

This system enabled automated metadata extraction from `.pbix` files, eliminating the need for manual inspection. It became a core component of a Power BI rationalisation tool, significantly improving efficiency and scalability.

---

## Project Link

🔗 **GitHub Repository:** https://github.com/Abhi-shekh/PBI-Rationalisation-Tool

## Code Snippet

```python
# Detect format
has_layout = "Report/Layout" in entries
has_definition = any(e.startswith("Report/definition/pages/") for e in entries)

if has_definition:
    _parse_format_b(zf, meta)
elif has_layout:
    _parse_format_a(zf, meta)


