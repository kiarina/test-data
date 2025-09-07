# kiarina/test-data

This repository is dedicated to distributing **shared test datasets** for multiple projects.
The actual data files are **not committed** to the repository history.
Instead, they are published as [GitHub Release assets](https://github.com/kiarina/test-data/releases).

---

## 📦 How to Download Data

### Using GitHub CLI

```sh
mkdir -p tests/data/large
gh release download --repo kiarina/test-data v2025.09 -p kiarina-python-dataset-v1.0.tar.zst --dir tests/data/large
tar --use-compress-program=unzstd -xvf tests/data/large/kiarina-python-dataset-v1.0.tar.zst -C tests/data/large
rm tests/data/large/kiarina-python-dataset-v1.0.tar.zst
```

### Using curl / wget

You can copy the asset download link directly from the [Releases](https://github.com/kiarina/test-data/releases) page.

```sh
# Example: download a dataset from release v2025.09
curl -L -o kiarina-python-dataset-v1.0.tar.zst \
  https://github.com/kiarina/test-data/releases/download/v2025.09/kiarina-python-dataset-v1.0.tar.zst
```

---

## 🔑 Versioning Policy

* Versions follow the format `vYYYY.MM` or `vYYYY.MM.DD`.
* Each release includes:
  * one or more dataset archives (`*.tar.zst`)
  * a checksum file (`SHA256SUMS`)
  * optionally, a `MANIFEST.md` describing the contents
* Each consuming project should **pin the dataset version** explicitly.

---

## 🗂 Example Release Layout

```
v2025.09/
  ├─ kiarina-python-dataset-v1.0.tar.zst
  ├─ SHA256SUMS
  └─ MANIFEST.md   # file descriptions
```

---

## ⚡ GitHub Actions Example

Here is a sample workflow that downloads the dataset, extracts it, and caches it across jobs.

```yaml
name: Test with Shared Data

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Restore dataset cache if available
      - name: Cache test dataset
        id: cache-dataset
        uses: actions/cache@v4
        with:
          path: tests/data/large
          key: test-data-v2025.09

      # Download dataset only if cache is missing
      - name: Download dataset
        if: steps.cache-dataset.outputs.cache-hit != 'true'
        run: |
          mkdir -p tests/data/large
          curl -L -o tests/data/large/kiarina-python-dataset-v1.0.tar.zst \
            https://github.com/kiarina/test-data/releases/download/v2025.09/kiarina-python-dataset-v1.0.tar.zst

      # Extract dataset into the cache directory
      - name: Extract dataset
        if: steps.cache-dataset.outputs.cache-hit != 'true'
        run: |
          tar --use-compress-program=unzstd -xvf tests/data/large/kiarina-python-dataset-v1.0.tar.zst -C tests/data/large
          rm tests/data/large/kiarina-python-dataset-v1.0.tar.zst

      # Run your tests with the dataset
      - name: Run tests
        run: |
          pytest --dataset-path tests/data/large
```

Notes:

* `actions/cache` keeps the dataset for up to **7 days of inactivity** and **10 GB per repository**.
* Use a **versioned cache key** (`test-data-v2025.09`) to ensure reproducibility.
* Always verify integrity with `sha256sum -c SHA256SUMS` if security matters.

---

## ⚖️ License

This repository itself is licensed under [MIT](./LICENSE).
The license terms of individual datasets are documented in their respective `MANIFEST.md` files.
