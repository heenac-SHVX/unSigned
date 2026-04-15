<img width="2500" height="351" alt="Immutaverse+Blue+White+Logo" src="https://github.com/user-attachments/assets/f8929d7f-94a1-43bf-b73f-29aff0fc3756" />

# Immutaverse Firmware Signing — Setup Guide

If you can read this, Immutaverse has granted you collaborator access to this repository. This guide will walk you through everything you need to set up firmware signing for your repository.

Once everything is set up, the process is simple: you commit a `.bin` firmware file with a specific commit message, and the signed firmware is automatically pushed back to your repository.

---

## Before You Start

Make sure you have the following ready:

- **OpenSSL** installed on your local machine
  - Linux / macOS: usually pre-installed. Verify by running `openssl version` in your terminal
  - Windows: install via [Git for Windows](https://gitforwindows.org/)
- Your **own GitHub repository** where you will commit firmware files and receive signed firmware back

---

## Step 1 — Add the Workflow File

The workflow file is already available in this repository. Here is how to copy it into your own repository:

1. In this repository, navigate to `.github/workflows/blank.yml` — or open it directly [here](https://github.com/immutaverse/Firmware-Signing/blob/main/.github/workflows/blank.yml)
2. Click the **Copy raw file** button (top-right of the file view) to copy the entire contents

Now add it to your own repository:

3. Go to your repository and click the **Actions** tab
4. Click **New workflow**
5. Click **"Set up a workflow yourself"** (top-right link)
6. Delete all the placeholder content in the editor
7. Paste the contents you copied
8. Name the file `firmware-signing.yml`
9. Click **Commit changes**

The file is now saved under `.github/workflows/` in your repository — GitHub handles this location automatically when you create it through the Actions UI.

---

## Step 2 — Generate Your Signing Keys

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
| `encrypted_signing_key` | **You will use this as your `SIGN_KEY` secret in Step 3** |

> Immutaverse uses `aes.key` and `iv.key` to decrypt your signing key when processing firmware. Without these, signing cannot proceed.

---

## Step 3 — Add Secrets to Your Repository

Your repository needs two secrets configured so the workflow can authenticate and sign firmware securely.

To add a secret, go to your repository on GitHub and navigate to:
**Settings → Secrets and variables → Actions → New repository secret**

---

### Secret 1: `IMT_TOKEN`

This is your own GitHub Personal Access Token (PAT). It allows the workflow to read your firmware file and push the signed firmware back to your repository.

**How to create your PAT:**

1. Click your profile picture (top-right on GitHub) → **Settings**
2. Scroll to the bottom of the left sidebar → click **Developer settings**
3. Click **Personal access tokens** → **Fine-grained tokens**
4. Click **Generate new token**
5. Enter a name such as `IMT_TOKEN`
6. Under **Repository access**, select **Only select repositories**, then use the dropdown to select your repository
7. Under **Permissions**, click **Add permissions**, search for **`contents`**, and set access to **Read and write**
8. Click **Generate token**
9. **Copy the token immediately** — GitHub will not show it again

Now add it as a secret:
- **Name:** `IMT_TOKEN`
- **Value:** The token you just copied

---

### Secret 2: `SIGN_KEY`

This is the encrypted signing key you generated in Step 2.

To get the value, open the `encrypted_signing_key` file in a text editor, or run the following in your terminal:

```bash
cat encrypted_signing_key
```

Copy the entire output, then add it as a secret:
- **Name:** `SIGN_KEY`
- **Value:** The contents of the `encrypted_signing_key` file

---

### Adding Both Secrets to Your Repository

Now that you have both values ready, add them to your repository:

1. Go to your repository on GitHub
2. Click **Settings** in the top navigation bar
3. In the left sidebar, click **Secrets and variables**
4. Click **Actions**
5. Scroll down until you find the **Repository secrets** section
6. Click **New repository secret**
7. Enter `IMT_TOKEN` as the name and paste your Personal Access Token as the value, then click **Add secret**
8. Click **New repository secret** again
9. Enter `SIGN_KEY` as the name and paste the contents of your `encrypted_signing_key` file as the value, then click **Add secret**

---

## Step 4 — Sign Your Firmware

Your setup is now complete. Every time you want to sign a firmware file, follow these steps:

1. Go to your repository on GitHub and navigate to the folder where you want to upload your firmware
2. Click **Add file** → **Upload files**
3. Drag and drop your `.bin` firmware file, or click **Choose your files** to select it
4. In the **Commit changes** section, enter the commit message **exactly** as shown:

   `Unsigned Firmware`

   > The commit message must be exactly `Unsigned Firmware`. Any variation will prevent the workflow from triggering.

5. Click **Commit changes**
6. Go to the **Actions** tab in your repository — you will see the workflow running
7. Once the workflow completes, reload the page — the signed firmware will appear in the same folder with `signed_` added to the beginning of the filename

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
