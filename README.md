Here’s a cleaned-up version of your Markdown, with consistent spacing, formatting, and proper Markdown conventions (e.g. code blocks, section spacing, and headers):

---

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

## 7. License

This project is licensed under the MIT License – see the `LICENSE` file for details.

---

Let me know if you'd like to add example outputs or CI/CD integration steps.
