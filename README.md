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
Great. Here's how to enhance your GitHub Actions setup to:

1. ✅ Generate SBOMs (Software Bill of Materials) for each package using Trivy.
2. ✅ Create a proper Debian and RPM repository layout.
3. ✅ Publish both repositories (and the SBOMs) to GitHub Pages under `gh-pages`.

---

## 🗂 Project Layout (after build)

```
├── repo/
│   ├── deb/
│   │   ├── pool/
│   │   │   └── my-app_1.0.0-1_amd64.deb
│   │   └── dists/
│   │       └── stable/
│   │           └── main/
│   │               └── binary-amd64/Packages.gz
│   ├── rpm/
│   │   ├── my-app-1.0.0-1.amd64.rpm
│   │   └── repodata/
│   │       └── (auto-generated by createrepo)
│   └── sbom/
│       ├── my-app_1.0.0-1_amd64.spdx.json
│       └── my-app-1.0.0-1.amd64.spdx.json
```

---

## ✅ GitHub Action: Full Workflow

### `.github/workflows/package-publish.yml`

```yaml
name: Package, Scan and Publish Repos

on:
  push:
    branches:
      - main

jobs:
  package-and-publish:
    runs-on: ubuntu-latest

    env:
      PACKAGE_NAME: my-app
      VERSION: 1.0.0
      BUILD_NUMBER: 1
      ARCH: amd64

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install prerequisites
        run: |
          sudo apt-get update
          sudo apt-get install -y ruby ruby-dev build-essential rpm dpkg-dev createrepo-c gzip
          sudo gem install --no-document fpm

      - name: Install Trivy
        run: |
          curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

      - name: Create output directories
        run: |
          mkdir -p repo/deb/pool
          mkdir -p repo/rpm
          mkdir -p repo/sbom

      - name: Build .deb
        run: |
          fpm -s dir -t deb \
              -n "$PACKAGE_NAME" \
              -v "$VERSION" \
              --iteration "$BUILD_NUMBER" \
              --architecture "$ARCH" \
              --description "Example package" \
              --prefix /usr/local/bin \
              my-app-binaries/

          mv *.deb repo/deb/pool/

      - name: Build .rpm
        run: |
          fpm -s dir -t rpm \
              -n "$PACKAGE_NAME" \
              -v "$VERSION" \
              --iteration "$BUILD_NUMBER" \
              --architecture "$ARCH" \
              --description "Example package" \
              --prefix /usr/local/bin \
              my-app-binaries/

          mv *.rpm repo/rpm/

      - name: Generate SBOMs
        run: |
          trivy sbom --format spdx-json --output "repo/sbom/${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.spdx.json" "repo/deb/pool/${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.deb"
          trivy sbom --format spdx-json --output "repo/sbom/${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.spdx.json" "repo/rpm/${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.rpm"

      - name: Generate Debian package index
        run: |
          mkdir -p repo/deb/dists/stable/main/binary-amd64
          dpkg-scanpackages repo/deb/pool /dev/null | gzip -9c > repo/deb/dists/stable/main/binary-amd64/Packages.gz

      - name: Generate RPM repo metadata
        run: |
          createrepo_c repo/rpm

      - name: Publish to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./repo
```

---

## 🔗 After Deployment

GitHub Pages will serve your repo at:

* **Debian:** `https://<username>.github.io/<repo>/deb/`
* **RHEL:** `https://<username>.github.io/<repo>/rpm/`
* **SBOMs:** `https://<username>.github.io/<repo>/sbom/`

You can now add the Debian repo like this:

```bash
echo "deb [trusted=yes] https://<username>.github.io/<repo>/deb stable main" | sudo tee /etc/apt/sources.list.d/my-app.list
sudo apt update
sudo apt install my-app
```

And for RPM:

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/my-app.repo
[my-app]
name=My App RPM Repo
baseurl=https://<username>.github.io/<repo>/rpm
enabled=1
gpgcheck=0
EOF

sudo yum install my-app
```

Excellent. To GPG-sign your Debian repository and add a signed `Release.gpg`, you’ll need to:

1. 🔐 Generate a GPG key (or use an existing one).
2. 🔏 Use it to sign your `Release` file.
3. 🧾 Include both `Release` and `Release.gpg` in the GitHub Pages repo.
4. ✅ Distribute the GPG public key so users can verify your repo.

---

## ✅ Step-by-Step GitHub Actions Integration

This setup uses GPG key material stored in GitHub Secrets:

| Secret Name           | Description                                                                     |
| --------------------- | ------------------------------------------------------------------------------- |
| `DEB_GPG_PRIVATE_KEY` | ASCII-armored private key (starts with `-----BEGIN PGP PRIVATE KEY BLOCK-----`) |
| `DEB_GPG_KEY_ID`      | Your GPG key ID or email (e.g. `myemail@example.com`)                           |
| `DEB_GPG_PASSPHRASE`  | Passphrase (if any) for the private key                                         |

---

### 📄 Workflow Additions (just after `dpkg-scanpackages`)

Add these steps **after** creating `Packages.gz` but **before** publishing to Pages:

```yaml
      - name: Import GPG key
        run: |
          echo "$DEB_GPG_PRIVATE_KEY" | gpg --batch --import
        env:
          DEB_GPG_PRIVATE_KEY: ${{ secrets.DEB_GPG_PRIVATE_KEY }}

      - name: Generate Release file
        run: |
          cd repo/deb
          cat > dists/stable/Release <<EOF
Origin: My App
Label: My App Debian Repo
Suite: stable
Codename: stable
Architectures: amd64
Components: main
Date: $(date -Ru)
Description: Signed Debian Repo for My App
EOF

      - name: Sign Release file (cleartext and detached)
        run: |
          cd repo/deb/dists/stable
          gpg --batch --yes --passphrase "$DEB_GPG_PASSPHRASE" \
              -u "$DEB_GPG_KEY_ID" -abs -o Release.gpg Release
          gpg --batch --yes --passphrase "$DEB_GPG_PASSPHRASE" \
              -u "$DEB_GPG_KEY_ID" --clearsign -o InRelease Release
        env:
          DEB_GPG_PASSPHRASE: ${{ secrets.DEB_GPG_PASSPHRASE }}
          DEB_GPG_KEY_ID: ${{ secrets.DEB_GPG_KEY_ID }}
```

---

## 🧑‍💻 Client Setup Instructions (Debian)

Distribute your **public key** via GitHub Pages or GPG servers:

```bash
# Download and import GPG public key
curl -fsSL https://<username>.github.io/<repo>/my-app.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/my-app.gpg > /dev/null

# Add signed repo
echo "deb https://<username>.github.io/<repo>/deb stable main" | sudo tee /etc/apt/sources.list.d/my-app.list

# Update and install
sudo apt update
sudo apt install my-app
```

---

## 🛠 How to Generate Your GPG Key Locally

```bash
gpg --full-generate-key  # Use RSA 4096, non-expiring, no comment

# Export public and private keys
gpg --armor --export-secret-keys "<your-email>" > private.asc
gpg --armor --export "<your-email>" > my-app.gpg
```

Then:

* Add `private.asc` to `DEB_GPG_PRIVATE_KEY` (GitHub Secret)
* Add `my-app.gpg` to your GitHub Pages `repo/` folder

---

Excellent. Signing the SBOMs (Software Bill of Materials) provides strong provenance and integrity guarantees, especially useful for air-gapped environments or regulated supply chains.

You’ll:

1. ✅ Generate and sign SBOMs (`.spdx.json`)
2. ✅ Publish both the SBOM and detached signature (`.asc`)
3. ✅ Allow clients to verify integrity using your GPG public key

---

## 🔐 GitHub Secrets Needed

You can reuse the **Debian** GPG key secrets:

| Secret Name           | Used For            |
| --------------------- | ------------------- |
| `DEB_GPG_PRIVATE_KEY` | Already used        |
| `DEB_GPG_KEY_ID`      | Same GPG key email  |
| `DEB_GPG_PASSPHRASE`  | If key is encrypted |

---

## 📄 Updated GitHub Action Step for SBOM Signing

Replace the SBOM generation step with:

```yaml
      - name: Generate and Sign SBOMs
        run: |
          # Generate SBOMs
          trivy sbom --format spdx-json --output "repo/sbom/${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.spdx.json" "repo/deb/pool/${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.deb"
          trivy sbom --format spdx-json --output "repo/sbom/${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.spdx.json" "repo/rpm/${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.rpm"

          # Import GPG key
          echo "$DEB_GPG_PRIVATE_KEY" | gpg --batch --import

          # Sign SBOMs
          gpg --batch --yes --passphrase "$DEB_GPG_PASSPHRASE" \
              -u "$DEB_GPG_KEY_ID" \
              -b -a "repo/sbom/${PACKAGE_NAME}_${VERSION}-${BUILD_NUMBER}_${ARCH}.spdx.json"

          gpg --batch --yes --passphrase "$DEB_GPG_PASSPHRASE" \
              -u "$DEB_GPG_KEY_ID" \
              -b -a "repo/sbom/${PACKAGE_NAME}-${VERSION}-${BUILD_NUMBER}.${ARCH}.spdx.json"
        env:
          DEB_GPG_PRIVATE_KEY: ${{ secrets.DEB_GPG_PRIVATE_KEY }}
          DEB_GPG_PASSPHRASE: ${{ secrets.DEB_GPG_PASSPHRASE }}
          DEB_GPG_KEY_ID: ${{ secrets.DEB_GPG_KEY_ID }}
```

---

## ✅ Resulting Files (in `repo/sbom/`)

```
my-app_1.0.0-1_amd64.spdx.json
my-app_1.0.0-1_amd64.spdx.json.asc
my-app-1.0.0-1.amd64.spdx.json
my-app-1.0.0-1.amd64.spdx.json.asc
```

---

## 🔍 Verifying SBOM Signatures (Client-Side)

```bash
# Import the public GPG key
curl -fsSL https://<username>.github.io/<repo>/my-app.gpg | gpg --import

# Verify the SBOM
gpg --verify my-app_1.0.0-1_amd64.spdx.json.asc my-app_1.0.0-1_amd64.spdx.json
```

Excellent choice. Embedding the SBOM inside the package ensures provenance and traceability **travels with the artifact**—even offline or in downstream mirrors.

Here’s how to do that in your GitHub Actions workflow.

---

## ✅ High-Level Plan

1. Generate SBOMs before packaging
2. Copy SBOMs into a doc path (e.g. `/usr/share/doc/<pkg>/`)
3. Adjust the `fpm` invocation to include that doc directory
4. Optionally sign external copies of SBOMs for GitHub Pages as before

---

## 🧱 Step-by-Step Adjustments

### 1. Generate SBOMs Early

Before you call `fpm`, generate the SBOMs **into a temporary local directory**:

```yaml
      - name: Generate SBOMs
        run: |
          mkdir -p ./sbom
          trivy sbom --format spdx-json --output ./sbom/my-app.deb.spdx.json my-app-binaries/
          trivy sbom --format spdx-json --output ./sbom/my-app.rpm.spdx.json my-app-binaries/
```

---

### 2. Embed SBOMs into `/usr/share/doc/<pkg>/`

Create a working structure before packaging:

```yaml
      - name: Prepare SBOM install path
        run: |
          mkdir -p staging/usr/local/bin
          cp -r my-app-binaries/* staging/usr/local/bin/

          mkdir -p staging/usr/share/doc/my-app
          cp ./sbom/my-app.deb.spdx.json staging/usr/share/doc/my-app/
          cp ./sbom/my-app.rpm.spdx.json staging/usr/share/doc/my-app/
```

---

### 3. Package with `fpm` (updated paths)

```yaml
      - name: Build .deb
        run: |
          fpm -s dir -t deb \
              -n "$PACKAGE_NAME" \
              -v "$VERSION" \
              --iteration "$BUILD_NUMBER" \
              --architecture "$ARCH" \
              --description "Example package with embedded SBOM" \
              --prefix / \
              -C staging \
              .
```

Same applies to the `.rpm` build.

---

### 4. Also Sign and Publish External SBOMs (optional)

If you still want to publish signed `.spdx.json` + `.asc` to GitHub Pages for direct download:

```yaml
      - name: Sign and publish SBOMs for web
        run: |
          mkdir -p repo/sbom
          cp ./sbom/* repo/sbom/

          echo "$DEB_GPG_PRIVATE_KEY" | gpg --batch --import

          for f in repo/sbom/*.spdx.json; do
            gpg --batch --yes --passphrase "$DEB_GPG_PASSPHRASE" \
                -u "$DEB_GPG_KEY_ID" \
                -b -a "$f"
          done
        env:
          DEB_GPG_PRIVATE_KEY: ${{ secrets.DEB_GPG_PRIVATE_KEY }}
          DEB_GPG_PASSPHRASE: ${{ secrets.DEB_GPG_PASSPHRASE }}
          DEB_GPG_KEY_ID: ${{ secrets.DEB_GPG_KEY_ID }}
```

---

## 📦 Resulting Package Contents

```text
/usr/local/bin/binary-one
/usr/share/doc/my-app/my-app.deb.spdx.json
```

(RPM contains the `.rpm.spdx.json` version)

---

## ✅ Client-Side SBOM Access

```bash
dpkg -L my-app | grep spdx
# or
rpm -ql my-app | grep spdx
```






