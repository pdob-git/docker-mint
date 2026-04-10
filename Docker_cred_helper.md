Here’s a **clean, minimal, and correct setup** for using the `pass` credential helper with **rootless Docker + GPG** on Linux (Mint/Ubuntu style).

No fluff — just what actually matters and works.

---

# ✅ 1. Install required tools

```bash
sudo apt update
sudo apt install -y pass gnupg2
```

Install Docker credential helper (latest binary):

```bash
wget https://github.com/docker/docker-credential-helpers/releases/latest/download/docker-credential-pass-v0.9.5-amd64
chmod +x docker-credential-pass-v0.9.5-amd64
mkdir -p ~/.local/bin
mv docker-credential-pass-v0.9.5-amd64 ~/.local/bin/docker-credential-pass
```

Ensure it’s in PATH:

```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

# 🔐 2. Create GPG key (for pass)

```bash
gpg --full-generate-key
```

Use:

* RSA 4096
* no expiration (or your choice)
* your email

List keys:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Example output:

```
sec   rsa4096/ABC123DEF4567890
```

👉 Copy the key ID:

```
ABC123DEF4567890
```

---

# 📦 3. Initialize pass store

```bash
pass init ABC123DEF4567890
```

This creates:

```
~/.password-store/
```

---

# 🐳 4. Configure Docker (rootless-safe)

Edit:

```bash
nano ~/.docker/config.json
```

**Clean config:**

```json
{
  "credsStore": "pass"
}
```

👉 That’s it. No extra junk.

---

# 🔄 5. Restart rootless Docker

```bash
systemctl --user restart docker
```

(Or log out/in if needed)

---

# 🔑 6. Test login

```bash
docker login
```

After login:

```bash
pass ls
```

You should see something like:

```
docker-credential-helpers/
└── https://index.docker.io/v1/
```

---

# 🧠 Important (rootless specifics)

✔ Works fully in rootless — no special config needed
✔ Uses your **user GPG key** (not system)
✔ Stored encrypted in:

```
~/.password-store
```

---

# ⚠️ Common mistakes (you likely hit one before)

❌ Installing helper via `go install` (broken path issue)
❌ Missing PATH → Docker can’t find helper
❌ Not restarting Docker after config change
❌ Using `credHelpers` instead of `credsStore` (wrong scope)

---

# 🔍 Verify everything

```bash
docker info | grep -i credential
```

Should show:

```
Credentials Store: pass
```

---

# 🚀 Optional hardening (recommended)

Lock GPG to require password each time:

```bash
gpg --edit-key ABC123DEF4567890
```

Then:

```
passwd
```

Also install pinentry:

```bash
sudo apt install pinentry-tty
```

---

# 🧾 Minimal final state

✔ `~/.docker/config.json`

```json
{ "credsStore": "pass" }
```

✔ `~/.password-store/` exists
✔ `docker-credential-pass` in PATH
✔ GPG key created

---

If you want, I can next:

* show **headless server setup (no GUI pinentry)**
* integrate this into your **OpenClaw / rootless Docker stack**
* or debug your current config step-by-step

Just tell me 👍
