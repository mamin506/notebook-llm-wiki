---
title: "Building for Production: From Bazel to Customers"
category: "patterns"
level: "intermediate"
status: "seedling"
sources: []
tags: ["#production", "#packaging", "#deployment", "#ci-cd", "#release", "#distribution"]
related: ["[[reference/calling-bazel-from-scripts]]", "[[concepts/advanced/hermeticity]]", "[[languages/cpp]]", "[[languages/python]]"]
last_updated: "2026-07-20"
---

# Building for Production: From Bazel to Customers

**Core insight:** Bazel is your *build system*, but shipping to customers requires orchestrating the full pipeline: build → package → test → sign → distribute.

---

## The Production Pipeline

```
Source Code
    ↓
  [BAZEL BUILD]  ← Hermetic compilation
    ↓
Artifacts in bazel-bin/
    ↓
  [PACKAGING]    ← Docker, native binary, DEB/RPM, etc.
    ↓
Packaged Product
    ↓
  [CI/CD TEST]   ← Run on packaged artifact
    ↓
Verified Product
    ↓
  [SIGN & UPLOAD] ← Code signing, checksums
    ↓
Distribution Server / Registry
    ↓
[CUSTOMER DOWNLOAD]
    ↓
Customer runs artifact
(No Bazel required!)
```

Your customers never see Bazel. They get a finished product.

---

## Step 1: Building with Bazel

### Output Location: bazel-bin/

When you run `bazel build //myapp:app`, the binary appears in:

```
bazel-bin/myapp/app          # For target //myapp:app
bazel-bin/mylib/libmylib.a   # For target //mylib:mylib (C++)
bazel-bin/py/app.pyz         # For py_binary (Python zip)
```

**Access from scripts:**

```bash
#!/bin/bash
OUTPUT_BASE=$(bazel info bazel-bin)
APP_BINARY="$OUTPUT_BASE/myapp/app"

# The binary is ready to run or package
"$APP_BINARY" --some-flag
```

### Artifact Types by Language

**C/C++:**
- `cc_binary` → ELF executable (Linux), Mach-O (macOS), PE (Windows)
- `cc_library` → `.a` static library or `.so` shared library
- Depends on compilation flags (`linkstatic`, etc.)

**Python:**
- `py_binary` → `.pyz` (Python zip archive with embedded interpreter)
- Self-contained; customer needs no Python interpreter
- Or: `py_library` compiled to `.so` extension modules (C++) for performance

**Java:**
- `java_binary` → Runnable `.jar` with manifest
- `java_library` → `.jar` class files

**Go:**
- `go_binary` → Static ELF executable (no runtime dependencies)

---

## Step 2: Packaging for Distribution

### Why Package?

Bazel outputs are raw artifacts. To ship to customers, you need:

1. **Portability** — Binaries depend on system libraries; package them with dependencies
2. **Integrity** — Checksums, signatures, version metadata
3. **Discoverability** — Where do customers get this? How do they know the version?
4. **Rollback** — Can customers easily downgrade if needed?

### Packaging Strategy 1: Native Binaries

**Best for:** Command-line tools, small applications, developers

```bash
#!/bin/bash
set -e

# Step 1: Build
bazel build //myapp:app -c opt

# Step 2: Extract binary
BINARY=$(bazel info bazel-bin)/myapp/app

# Step 3: Copy to distribution directory
mkdir -p dist/v1.0.0/linux-x86_64
cp "$BINARY" dist/v1.0.0/linux-x86_64/myapp
chmod +x dist/v1.0.0/linux-x86_64/myapp

# Step 4: Generate checksum
cd dist/v1.0.0/linux-x86_64
sha256sum myapp > myapp.sha256

# Step 5: Upload to server/GitHub releases
# gsutil cp dist/v1.0.0/* gs://my-releases/
```

**Release URL:** `https://releases.example.com/myapp/v1.0.0/linux-x86_64/myapp`

**Customer:** Downloads binary and runs directly
```bash
curl -o myapp https://releases.example.com/myapp/v1.0.0/linux-x86_64/myapp
chmod +x myapp
./myapp
```

### Packaging Strategy 2: Docker Container

**Best for:** Microservices, complex dependencies, cloud deployment

```bash
#!/bin/bash
set -e

# Step 1: Build binary with Bazel
bazel build //myapp:app -c opt

# Step 2: Create Dockerfile
cat > Dockerfile << 'EOF'
FROM gcr.io/distroless/base-debian11

COPY bazel-bin/myapp/app /app
ENTRYPOINT ["/app"]
EOF

# Step 3: Extract binary to Docker build context
mkdir -p docker_build
cp $(bazel info bazel-bin)/myapp/app docker_build/

# Step 4: Build Docker image
docker build -t gcr.io/mycompany/myapp:v1.0.0 -f Dockerfile docker_build/

# Step 5: Push to registry
docker push gcr.io/mycompany/myapp:v1.0.0
```

**Release:** Image in container registry

**Customer:** Runs with Docker
```bash
docker run gcr.io/mycompany/myapp:v1.0.0
```

### Packaging Strategy 3: System Packages (DEB/RPM)

**Best for:** Linux distributions, system-level integration, package managers

```bash
#!/bin/bash
set -e

# Step 1: Build
bazel build //myapp:app -c opt

# Step 2: Prepare package directory structure
mkdir -p pkg/usr/bin pkg/etc pkg/usr/share/doc

# Step 3: Copy binary and metadata
cp $(bazel info bazel-bin)/myapp/app pkg/usr/bin/
cp LICENSE pkg/usr/share/doc/myapp.LICENSE

# Step 4: Create control file (for DEB)
mkdir -p pkg/DEBIAN
cat > pkg/DEBIAN/control << 'EOF'
Package: myapp
Version: 1.0.0
Architecture: amd64
Maintainer: Your Company <support@example.com>
Description: My Application
EOF

# Step 5: Build DEB package
dpkg-deb --build pkg myapp_1.0.0_amd64.deb

# Step 6: Upload to package repository
# aws s3 cp myapp_1.0.0_amd64.deb s3://my-deb-repo/
```

**Release:** `.deb` in apt repository

**Customer:** Uses package manager
```bash
apt install myapp
```

### Packaging Strategy 4: Python Distribution (PyPI)

**Best for:** Python packages consumed by other Python projects

```bash
# BUILD.bazel
py_binary(
    name = "myapp",
    srcs = ["myapp.py"],
    deps = ["//lib:mylib"],
)

# setup.py (outside Bazel)
setup(
    name="myapp",
    version="1.0.0",
    py_modules=["myapp"],
    install_requires=[...],
    entry_points={
        "console_scripts": [
            "myapp=myapp:main",
        ],
    },
)

# Package it
python setup.py sdist bdist_wheel
twine upload dist/myapp-1.0.0.tar.gz
```

**Release:** On PyPI

**Customer:** Uses pip
```bash
pip install myapp
```

---

## Step 3: Testing the Packaged Artifact

**Critical:** Test the *packaged* artifact, not just the raw binary. Packaging can hide bugs.

```bash
#!/bin/bash
set -e

# Build
bazel build //myapp:app -c opt

# Create Docker image (example)
docker build -t myapp:v1.0.0 .

# Test the image (not the raw binary)
docker run myapp:v1.0.0 --version

# Integration tests on packaged artifact
docker run myapp:v1.0.0 --run-tests

# Performance tests
docker run --memory=512m myapp:v1.0.0 < large_input.txt
```

---

## Step 4: Code Signing & Verification

### For Native Binaries

**Linux (GPG):**
```bash
#!/bin/bash
gpg --detach-sign --armor myapp
# Creates myapp.asc (signature file)

# Customer verifies:
gpg --verify myapp.asc myapp
```

**macOS (Codesign):**
```bash
codesign -s "Developer ID" myapp
spctl -a -vvv -t install myapp  # Customer verifies
```

**Windows (Authenticode):**
```bash
signtool sign /f certificate.pfx /t http://timestamp.server myapp.exe
```

### For Checksums

```bash
# Build and sign
sha256sum myapp > myapp.sha256
gpg --detach-sign --armor myapp.sha256

# Customer verifies
sha256sum -c myapp.sha256
gpg --verify myapp.sha256.asc myapp.sha256
```

### For Container Images

Docker images include digest verification:

```bash
# Push with digest
docker push gcr.io/mycompany/myapp:v1.0.0
# → Prints: Digest: sha256:abc123...

# Customer pulls and verifies
docker pull gcr.io/mycompany/myapp:v1.0.0@sha256:abc123...
```

---

## Step 5: Distribution Channels

### Option A: Direct Download

**Website/GitHub Releases:**

```bash
#!/bin/bash
# Build & package
bazel build //myapp:app -c opt

# Upload to GitHub
gh release create v1.0.0 \
    --title "MyApp v1.0.0" \
    --notes "Bug fixes and performance improvements" \
    $(bazel info bazel-bin)/myapp/app
```

**Pros:** Simple, direct control  
**Cons:** No automatic updates, users must check manually

### Option B: Package Repository

**apt/yum/Homebrew:**

```bash
# DEB: Upload to apt repository
aws s3 sync ./deb-repo s3://my-apt-repo/

# RPM: Upload to yum repository
aws s3 sync ./rpm-repo s3://my-yum-repo/

# Homebrew: Create formula (template)
class Myapp < Formula
  url "https://releases.example.com/myapp/v1.0.0/myapp-v1.0.0.tar.gz"
  sha256 "abc123..."
  
  def install
    bin.install "myapp"
  end
end
```

**Pros:** Automatic updates via package manager, dependency management  
**Cons:** Requires maintenance of packages for multiple distros

### Option C: Container Registry

**Docker Hub, GCR, ECR, Artifactory:**

```bash
# Build & push to GCR
bazel build //myapp:container
docker tag myapp:latest gcr.io/mycompany/myapp:v1.0.0
docker push gcr.io/mycompany/myapp:v1.0.0

# Customer pulls
docker pull gcr.io/mycompany/myapp:v1.0.0
```

**Pros:** Language-agnostic, dependency isolation, easy scaling  
**Cons:** Overhead for lightweight tools

### Option D: Language-Specific Registries

**PyPI (Python), NPM (JavaScript), Maven Central (Java):**

```bash
# Python example
bazel build //myapp:wheel
python -m twine upload dist/myapp-1.0.0-py3-none-any.whl
```

**Pros:** Integrated with language ecosystem  
**Cons:** Specific to one language

---

## Step 6: Versioning & Releases

### Semantic Versioning

```
MAJOR.MINOR.PATCH  (e.g., 1.2.3)

MAJOR: Breaking changes
MINOR: New features, backward compatible
PATCH: Bug fixes
```

### Release Metadata

Embed version info in binaries at build time:

```starlark
# BUILD.bazel
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    copts = [
        "-DAPP_VERSION=1.2.3",
        "-DBUILD_DATE=$(date +%Y-%m-%d)",
        "-DBUILD_GIT_COMMIT=$(git rev-parse HEAD)",
    ],
)
```

**Output:**
```bash
$ ./app --version
MyApp v1.2.3 (built 2026-07-20, commit abc123)
```

**Customer can verify:**
- Version matches what they expect
- Binary is authentic (matches Git commit they trust)

---

## Step 7: Production CI/CD Pipeline

### Complete Example (GitHub Actions)

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - v*

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: bazel-contrib/setup-bazel@v0
      
      # Step 1: Build with Bazel
      - name: Build release binary
        run: |
          bazel build //myapp:app \
            -c opt \
            --bazelrc=/dev/null
      
      # Step 2: Extract and sign
      - name: Extract and sign binary
        env:
          GPG_SECRET_KEY: ${{ secrets.GPG_SECRET_KEY }}
        run: |
          BINARY=$(bazel info bazel-bin)/myapp/app
          mkdir -p dist/linux-x86_64
          cp "$BINARY" dist/linux-x86_64/myapp
          chmod +x dist/linux-x86_64/myapp
          
          echo "$GPG_SECRET_KEY" | gpg --import
          gpg --detach-sign --armor dist/linux-x86_64/myapp
          sha256sum dist/linux-x86_64/myapp > dist/linux-x86_64/myapp.sha256
      
      # Step 3: Create GitHub release
      - name: Create GitHub release
        run: |
          gh release create ${{ github.ref }} \
            --title "MyApp ${{ github.ref }}" \
            --notes "Release notes here" \
            dist/linux-x86_64/myapp \
            dist/linux-x86_64/myapp.asc \
            dist/linux-x86_64/myapp.sha256
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      # Step 4: Build and push Docker image
      - name: Build and push Docker image
        run: |
          docker build -t gcr.io/mycompany/myapp:${{ github.ref }} .
          docker push gcr.io/mycompany/myapp:${{ github.ref }}
      
      # Step 5: Push to package repository (optional)
      - name: Upload to APT repository
        run: |
          # Build DEB package
          ./scripts/build_deb.sh dist/linux-x86_64/myapp
          # Upload to S3
          aws s3 cp *.deb s3://my-apt-repo/
```

---

## Key Principles

### 1. Separation of Concerns

- **Bazel** handles: Compilation, dependency resolution, hermeticity
- **Packaging** handles: Distribution format, metadata, signatures
- **CI/CD** handles: Orchestration, automated testing, releasing

Don't try to do everything in Bazel.

### 2. Hermetic Builds for Reproducibility

```bash
# Clean build from scratch
bazel clean --expunge
bazel build //myapp:app -c opt --bazelrc=/dev/null

# Should produce identical binary (byte-for-byte) if inputs haven't changed
```

Reproducibility = customers can verify authenticity.

### 3. Test the Packaged Artifact

```bash
# ❌ Bad: Test only the raw binary
bazel test //...

# ✅ Good: Also test in packaged form
docker run myapp:test
apt install ./myapp.deb && myapp --test
```

Bugs hide in packaging!

### 4. Version Everything

- Binary version: embedded in artifact
- Docker image: tag with semantic version
- Package: DEB/RPM version field
- Git: tag the source commit (`git tag v1.2.3`)

Traceability from source → customer hands.

---

## Common Patterns by Company Size

### Small Company (Single Product)

```
Source (GitHub)
    ↓
GitHub Actions (Bazel build)
    ↓
GitHub Releases (native binary)
    ↓
Customers download + run
```

Minimal overhead; works for CLI tools, small apps.

### Medium Company (Multiple Services)

```
Monorepo (Bazel workspace)
    ↓
CI/CD (Bazel + Docker)
    ↓
Container Registry (GCR/ECR)
    ↓
Kubernetes deployment
    ↓
Customers use web app
```

Containers handle dependency isolation; Bazel handles build orchestration.

### Large Company (Platform & Ecosystem)

```
Monorepo (Bazel workspace)
    ↓
CI/CD (Bazel + multi-platform builds)
    ↓
Multiple Distribution Channels:
├─ Docker Registry (cloud customers)
├─ Package Repositories (Linux distros)
├─ GitHub Releases (open-source binaries)
└─ PyPI/NPM/Maven (language-specific)
    ↓
Customers choose their installation method
```

Bazel coordinates the build; packaging infrastructure handles distribution.

---

## See Also

- [[reference/calling-bazel-from-scripts]] — How to invoke Bazel in build scripts
- [[concepts/advanced/hermeticity]] — Why hermetic builds matter for production
- [[languages/cpp]], [[languages/python]] — Language-specific artifacts
- [[experiments/publishing-bazel-rules]] — Packaging Bazel rulesets (similar pattern)
