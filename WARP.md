# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a curated list of public APIs organized by category. It's a community-driven project that maintains a comprehensive, manually-curated collection of APIs across various domains. The repository does not contain API implementations—it's a documentation and discovery resource.

## Repository Structure

```
.
├── README.md                  # Main API catalog (single source of truth)
├── CONTRIBUTING.md           # Contribution guidelines
├── scripts/                  # Validation and testing infrastructure
│   ├── validate/            # Python validation package
│   │   ├── format.py        # Entry format validation
│   │   └── links.py         # Link validation and duplicate checking
│   ├── tests/               # Unit tests for validators
│   │   ├── test_validate_format.py
│   │   └── test_validate_links.py
│   ├── github_pull_request.sh  # CI/CD validation script
│   └── requirements.txt     # Python dependencies
└── .github/
    └── workflows/           # GitHub Actions CI/CD pipelines
```

## Common Development Tasks

### Validation

**Validate entry formatting:**
```bash
python scripts/validate/format.py README.md
```

**Validate links (full check - can take time):**
```bash
python scripts/validate/links.py README.md
```

**Check for duplicate links only (faster):**
```bash
python scripts/validate/links.py README.md --only_duplicate_links_checker
```
Short form:
```bash
python scripts/validate/links.py README.md -odlc
```

### Testing

**Install dependencies:**
```bash
python -m pip install -r scripts/requirements.txt
```

**Run all tests:**
```bash
cd scripts
python -m unittest discover tests/ --verbose
```

**Run format tests only:**
```bash
cd scripts
python -m unittest discover tests/ --verbose --pattern "test_validate_format.py"
```

**Run link tests only:**
```bash
cd scripts
python -m unittest discover tests/ --verbose --pattern "test_validate_links.py"
```

## API Entry Format

Each API entry must follow this exact format:

```markdown
| [API Title](https://api-docs-url) | Description (max 100 chars) | `apiKey` | Yes | Yes |
```

### Field Requirements

1. **Title**: 
   - Must NOT end with "API" (e.g., use "Gmail" not "Gmail API")
   - Must NOT include TLD (e.g., "Gmail" not "Gmail.com")
   - Wrapped in `[Title](URL)` markdown syntax

2. **Description**: 
   - First character must be capitalized
   - Must NOT end with punctuation (except for parentheses)
   - Maximum 100 characters
   - Must be descriptive but concise

3. **Auth** (valid values only):
   - `` `apiKey` `` - API uses private key/token
   - `` `OAuth` `` - API supports OAuth
   - `` `X-Mashape-Key` `` - Specific header name required
   - `` `User-Agent` `` - User-Agent header required
   - `No` - No authentication required
   - Auth values (except "No") must be enclosed in backticks

4. **HTTPS** (valid values):
   - `Yes` or `No`

5. **CORS** (valid values):
   - `Yes`, `No`, or `Unknown`

### Entry Constraints

- Entries within each category must be in **alphabetical order**
- Each table column must be padded with one space on either side
- Minimum 3 entries required to create a new category
- Only free or freemium APIs (not APIs requiring device purchase)

## Validation Architecture

### Format Validator (`scripts/validate/format.py`)

The format validator enforces structural and content rules:

- **Alphabetical ordering**: Checks that entries within each category are sorted
- **Title validation**: Ensures proper markdown link syntax and no "API" suffix
- **Description validation**: Checks capitalization, length (≤100 chars), and no trailing punctuation
- **Auth/HTTPS/CORS validation**: Verifies only accepted values are used
- **Category requirements**: Ensures minimum entry count per category

Key constants:
- `max_description_length = 100`
- `min_entries_per_category = 3`
- `num_segments = 5` (expected table columns)

### Link Validator (`scripts/validate/links.py`)

The link validator checks URL integrity:

- **Duplicate detection**: Finds duplicate URLs across the entire document
- **Link availability**: HTTP requests to verify links return valid responses
- **Cloudflare protection detection**: Handles special cases where Cloudflare protection returns 403/503 but link is valid
- **User-Agent spoofing**: Uses rotating user agents to avoid blocks
- **Timeout handling**: 25-second timeout for requests

Error codes:
- `ERR:CLT` - Client error (4xx status)
- `ERR:SSL` - SSL certificate error
- `ERR:CNT` - Connection error
- `ERR:TMO` - Timeout error
- `ERR:TMR` - Too many redirects
- `ERR:UKN` - Unknown error

## CI/CD Workflows

### On Push & Pull Request

**Workflow**: `.github/workflows/test_of_push_and_pull.yml`

- Validates README.md format
- On PR: validates only changed lines using `github_pull_request.sh`
- On Push: checks for duplicate links only (faster)

### Validation Package Tests

**Workflow**: `.github/workflows/test_of_validate_package.yml`

- Runs unit tests for validation scripts
- Ensures validator integrity

## Contributing Rules

When adding or modifying entries:

1. **One API per pull request**
2. **PR title format**: `Add [API-name] API`
3. **Commit message format**: `Add [API-name] API to [Category]`
4. **Squash all commits** before submitting PR
5. **Search existing PRs/issues** to avoid duplicates
6. **Alphabetical order** within categories
7. **API must have proper documentation**
8. **API must be free or have a free tier**
9. **No APIs requiring device purchase**

### Pull Request Checklist

- [ ] Formatted according to guidelines
- [ ] Ordered alphabetically
- [ ] Description ≤ 100 characters
- [ ] Description doesn't end with punctuation
- [ ] Each column padded with one space
- [ ] Searched for duplicates
- [ ] All commits squashed
- [ ] New categories have ≥ 3 items

## Python Requirements

The validation scripts require:
- Python 3.8+
- `requests` library
- `certifi`, `charset-normalizer`, `idna`, `urllib3` (dependencies)

## Notes

- This is a **catalog/documentation repository**, not an API implementation
- The `README.md` is the single source of truth for all API listings
- Validation is automated via GitHub Actions on all PRs
- The project prioritizes community curation over marketing attempts
- APIs are manually reviewed for quality and relevance
