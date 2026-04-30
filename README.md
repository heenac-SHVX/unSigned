# Immutaverse Firmware Signing

![Immutaverse Blue White Logo](https://github.com/user-attachments/assets/f8929d7f-94a1-43bf-b73f-29aff0fc3756)

![GitHub Marketplace](https://img.shields.io/badge/GitHub-Marketplace-blue?logo=github)
![Version](https://img.shields.io/github/v/release/adishvx/unSigned)
![License](https://img.shields.io/github/license/adishvx/unSigned)

Securely sign firmware binaries directly from GitHub Actions using the Immutaverse Firmware Signing Action.

---

# Overview

The Immutaverse Firmware Signing Action allows you to automatically sign firmware binaries (`.bin`) directly inside GitHub Actions workflows.

This action is designed for embedded developers, IoT pipelines, firmware release automation, and secure software delivery.

---

# Before You Begin

Make sure you have the following ready:

- **OpenSSL** installed on your local machine
  - Linux / macOS: usually pre-installed. Verify by running `openssl version`
  - Windows: install via [Git for Windows](https://gitforwindows.org/)
- Your **own GitHub repository** where firmware files will be uploaded and signed firmware will be returned

---


# Setup Guide

## Step 1 — Create the Workflow

Create the following file inside your repository:

```text
.github/workflows/firmware-signing.yml
```

Paste the following workflow:

```yaml
name: Sign Firmware

on:
  push:
    branches:
      - main
    paths:
      - '**/*.bin'

jobs:
  sign:
    if: github.event.head_commit.message == 'Unsigned Firmware'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Firmware Signing Action
        uses: adishvx/unSigned@<LATEST_VERSION>
        with:
          imt-token: ${{ secrets.IMT_TOKEN }}
          sign-key: ${{ secrets.SIGN_KEY }}
```

---

# Repository Secrets

Your repository requires two GitHub Actions secrets.

Navigate to:

```text
Repository → Settings → Secrets and variables → Actions
```

---

## Secret 1 — IMT_TOKEN

A GitHub Fine-grained Personal Access Token (PAT) is required to:

- Read firmware files
- Push signed firmware back to the repository
- Authenticate with Immutaverse services

### Create a GitHub PAT

1. Open GitHub Settings
2. Go to **Developer settings**
3. Open **Personal access tokens**
4. Select **Fine-grained tokens**
5. Click **Generate new token**
6. Name the token:

```text
IMT_TOKEN
```

7. Under **Repository access**, select:

```text
Only select repositories
```

8. Choose your repository
9. Under **Repository permissions**, enable:

| Permission | Access |
|------------|--------|
| Contents | Read and write |

10. Under **Account permissions**, enable:

| Permission | Access |
|------------|--------|
| Email addresses | Read-only |

11. Generate the token
12. Copy the token immediately

GitHub will not display it again.

### Add IMT_TOKEN Secret

1. Open Repository Settings
2. Go to **Secrets and variables → Actions**
3. Click **New repository secret**
4. Name it:

```text
IMT_TOKEN
```

5. Paste your PAT value
6. Click **Add secret**

---

## Secret 2 — SIGN_KEY

This is your private Immutaverse firmware signing key.

### Add SIGN_KEY Secret

1. Open Repository Settings
2. Go to **Secrets and variables → Actions**
3. Click **New repository secret**
4. Name it:

```text
SIGN_KEY
```

5. Paste your signing key
6. Click **Add secret**

---

# Sign Firmware

Once setup is complete:

1. Open your GitHub repository
2. Upload a firmware `.bin` file
3. Use the following commit message exactly:

```text
Unsigned Firmware
```

4. Commit the upload
5. Open the **Actions** tab
6. Wait for workflow completion
7. Refresh the repository folder

The signed firmware will appear automatically.

### Example

```text
firmware.bin
↓
signed_firmware.bin
```

---

# Important Requirements

- Firmware files must use the `.bin` extension
- Workflow only runs on the `main` branch
- Commit message must exactly match:

```text
Unsigned Firmware
```

---


# Support

For signing issues, licensing, or subscription inquiries, visit:

https://immutaverse.com

---

# License

Copyright © Immutaverse Inc.

This software is proprietary. Unauthorized redistribution, modification, or resale is prohibited.
