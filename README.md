# Packaging with `fpm` and Scanning with `Trivy`

This guide provides a step-by-step workflow on how to package binaries into `.deb` and `.rpm` formats using [`fpm`](https://fpm.readthedocs.io/), and how to scan the artifacts for vulnerabilities using [`Trivy`](https://aquasecurity.github.io/trivy/).

---

## Table of Contents

1. [Install Prerequisites](#1-install-prerequisites)
2. [Prepare Your Binaries](#2-prepare-your-binaries)
3. [Scan Binaries Before Packaging (Recommended)](#3-recommended-scan-binaries-before-packaging)
4. [Create the `.deb` Package](#4-create-the-deb-package)
5. [Create the `.rpm` Package](#5-create-the-rpm-package)
6. [Final Verification Scan (Optional)](#6-optional-final-verification-scan-of-packages)
7. [License](#7-license)

---

## 1. Install Prerequisites

Before you begin, install `fpm` and `Trivy`.

### a. Install `fpm`

`fpm` is a Ruby gem, so you'll need Ruby and `gem` installed.

**On Debian/Ubuntu:**

```bash
sudo apt-get update
sudo apt-get install ruby ruby-dev build-essential
```

**On RHEL/CentOS/Fedora:**

```bash
sudo yum install ruby ruby-devel rpm-build
```

Then install `fpm`:

```bash
sudo gem install fpm
```

### b. Install Trivy

Install Trivy using the official install script:

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
```

For other methods, see the [official Trivy documentation](https://aquasecurity.github.io/trivy/).

---

## 2. Prepare Your Binaries

Gather the compiled binaries into a dedicated directory, e.g.:

```
my-app-binaries/
├── binary-one
├── binary-two
└── binary-three
```

---

## 3. (Recommended) Scan Binaries Before Packaging

Use Trivy to scan the raw binaries for known vulnerabilities.

```bash
# Navigate to the parent directory of your binaries
cd /path/to/

# Scan the directory
trivy fs ./my-app-binaries/
```

Review and address any critical vulnerabilities before proceeding.

---

## 4. Create the `.deb` Package

Package the binaries using `fpm`.

```bash
cd /path/to/my-app-binaries/

fpm -s dir -t deb \
    -n <package-name> \
    -v <version> \
    --iteration <build-number> \
    --architecture <architecture> \
    --description "A description for your package." \
    --prefix /usr/local/bin \
    .
```

---

## 5. Create the `.rpm` Package

The RPM creation process is nearly identical:

```bash
cd /path/to/my-app-binaries/

fpm -s dir -t rpm \
    -n <package-name> \
    -v <version> \
    --iteration <build-number> \
    --architecture <architecture> \
    --description "A description for your package." \
    --prefix /usr/local/bin \
    .
```

---

## 6. (Optional) Final Verification Scan of Packages

As a final step, scan the generated packages.

**Scan the `.deb` package:**

```bash
trivy fs <package-name>_<version>-<build-number>_<architecture>.deb
```

**Scan the `.rpm` package:**

```bash
trivy fs <package-name>-<version>-<build-number>.<architecture>.rpm
```

After running these checks, your packages should be ready for distribution.

---
Here’s a GitHub Actions workflow that:

1. Installs dependencies (`Ruby`, `fpm`, and `Trivy`)
2. Packages your binaries into `.deb` and `.rpm` formats
3. Scans the output artifacts with Trivy
4. Uploads the results as GitHub Actions artifacts

---

### 📄 `.github/workflows/package-and-scan.yml`

```yaml
name: Package and Scan Binaries

on:
  push:
    paths:
      - 'my-app-binaries/**'
      - '.github/workflows/package-and-scan.yml'
    branches:
      - main

jobs:
  build-and-scan:
    runs-on: ubuntu-latest

    env:
      PACKAGE_NAME: my-app
      VERSION: 1.0.0
      BUILD_NUMBER: 1
      ARCH: amd64

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Ruby and install fpm
        run: |
          sudo apt-get update
          sudo apt-get install -y ruby ruby-dev build-essential rpm
          sudo gem install --no-document fpm

      - name: Install Trivy
        run: |
          curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

      - name: Create .deb package
        run: |
          fpm -s dir -t deb \
              -n "$PACKAGE_NAME" \
              -v "$VERSION" \
              --iteration "$BUILD_NUMBER" \
              --architecture "$ARCH" \
              --description "Example package created via GitHub Actions." \
              --prefix /usr/local/bin \
              my-app-binaries/

      - name: Create .rpm package
        run: |
          fpm -s dir -t rpm \
              -n "$PACKAGE_NAME" \
              -v "$VERSION" \
              --iteration "$BUILD_NUMBER" \
              --architecture "$ARCH" \
              --description "Example package created via GitHub Actions." \
              --prefix /usr/local/bin \
              my-app-binaries/

      - name: Scan .deb package with Trivy
        run: |
          trivy fs "${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.deb"

      - name: Scan .rpm package with Trivy
        run: |
          trivy fs "${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.rpm"

      - name: Upload .deb and .rpm packages
        uses: actions/upload-artifact@v4
        with:
          name: packaged-artifacts
          path: |
            *.deb
            *.rpm
```

---

### 📌 Notes:

* Replace `my-app-binaries/` with the correct path to your binary folder if different.
* You can parameterise the `VERSION`, `PACKAGE_NAME`, etc., via a file or input later.
* The workflow assumes your binaries are ready and don't need to be built from source.


---


