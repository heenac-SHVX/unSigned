<img width="2500" height="351" alt="Immutaverse+Blue+White+Logo" src="https://github.com/user-attachments/assets/f8929d7f-94a1-43bf-b73f-29aff0fc3756" />


# Immutaverse Firmware Signing — GitHub Actions Setup Guide

After purchasing a subscription, Immutaverse will add your GitHub handle as a collaborator on the Immutaverse repository. This gives you access to the workflow file needed for firmware signing.

Once everything is set up, the process is simple: you commit a `.bin` firmware file with a specific commit message, and the signed firmware is automatically pushed back to your repository.

---

## Before You Start

Make sure you have the following ready:

- A **GitHub account**
- **OpenSSL** installed on your local machine
  - Linux / macOS: usually pre-installed. Verify by running `openssl version` in your terminal
  - Windows: install via [Git for Windows](https://gitforwindows.org/)
- Your **GitHub username sent to Immutaverse** so they can grant you collaborator access

---

## Step 1 — Create a New GitHub Repository

Create a new repository in your GitHub account. This will be the repository where you commit firmware files and where signed firmware gets returned.

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** icon (top-right) → **New repository**
3. Give it a name, set it to **Private**, and click **Create repository**

---

## Step 2 — Add the Workflow File

Immutaverse will provide you with a workflow file via the shared repository you now have collaborator access to. Copy that file into your repository:

1. In your repository, click the **Actions** tab
2. Click **New workflow**
3. Click **"Set up a workflow yourself"** (top-right link)
4. Delete all the placeholder content in the editor
5. Paste the contents of the workflow file provided by Immutaverse
6. Name the file `firmware-signing.yml`
7. Click **Commit changes**

The file is now saved under `.github/workflows/` in your repository — GitHub handles this location automatically when you create it through the Actions UI.

---

## Step 3 — Generate Your Signing Keys

You need to generate a private signing key, encrypt it, and share the encryption details with Immutaverse. Run the following commands in your terminal:

```bash
# Generate RSA private key
openssl genrsa -out signing_key.pem 3072

# Generate a random AES encryption key and IV
AES=$(openssl rand -hex 32)
IV=$(openssl rand -hex 16)

# Save the AES key and IV to files
echo "$AES" > aes.key
echo "$IV" > iv.key

# Encrypt the private key using AES-256-CBC
openssl enc -aes-256-cbc \
  -in signing_key.pem \
  -out output.enc \
  -K "$AES" \
  -iv "$IV"

# Base64-encode the encrypted key
base64 -w0 output.enc > encrypted_signing_key
```

Once the commands finish, you will have these files:

| File | What to do with it |
|---|---|
| `signing_key.pem` | **Keep this private. Never share or commit it.** Store it securely offline. |
| `aes.key` | **Send to Immutaverse** via email or the agreed channel |
| `iv.key` | **Send to Immutaverse** via email or the agreed channel |
| `encrypted_signing_key` | **You will use this as your `SIGN_KEY` secret in Step 4** |

> Immutaverse uses `aes.key` and `iv.key` to decrypt your signing key when processing firmware. Without these, signing cannot proceed.

---

## Step 4 — Add Secrets to Your Repository

Your repository needs two secrets configured so the workflow can authenticate and sign firmware securely.

To add a secret, go to your repository on GitHub and navigate to:
**Settings → Secrets and variables → Actions → New repository secret**

---

### Secret 1: `IMT_TOKEN`

This is your own GitHub Personal Access Token (PAT). It allows the workflow to read your firmware file and push the signed firmware back to your repository.

**How to create your PAT:**

1. Click your profile picture (top-right on GitHub) → **Settings**
2. Scroll to the bottom of the left sidebar → click **Developer settings**
3. Click **Personal access tokens** → **Tokens (classic)**
4. Click **Generate new token (classic)**
5. Enter a name such as `Immutaverse Firmware Signing`
6. Under **Select scopes**, check the **`repo`** checkbox (this grants full read/write access to your repositories)
7. Click **Generate token**
8. **Copy the token immediately** — GitHub will not show it again

Now add it as a secret:
- **Name:** `IMT_TOKEN`
- **Value:** The token you just copied

---

### Secret 2: `SIGN_KEY`

This is the encrypted signing key you generated in Step 3.

To get the value, open the `encrypted_signing_key` file in a text editor, or run the following in your terminal:

```bash
cat encrypted_signing_key
```

Copy the entire output, then add it as a secret:
- **Name:** `SIGN_KEY`
- **Value:** The contents of the `encrypted_signing_key` file

---

## Step 5 — Sign Your Firmware

Your setup is now complete. Every time you want to sign a firmware file, follow these steps:

1. Add your `.bin` firmware file to the repository
2. Commit and push it with the commit message **exactly** as shown below:

```bash
git add your_firmware.bin
git commit -m "Unsigned Firmware"
git push
```

> The commit message must be exactly `Unsigned Firmware`. If the message is different, the workflow will not trigger.

3. Go to the **Actions** tab in your repository — you will see the workflow running
4. Once the workflow completes, the signed firmware will appear in the same folder with `signed_` added to the beginning of the filename

**Example:**
```
your_firmware.bin   →   signed_your_firmware.bin
```

---

## Important Notes

- The firmware file must have a **`.bin`** extension
- The push must be to the **`main`** branch
- The commit message must be **exactly** `Unsigned Firmware` — any variation will be ignored
- `aes.key` and `iv.key` must be shared with Immutaverse before your first signing run
- Never commit `signing_key.pem` to any repository — store it securely offline
