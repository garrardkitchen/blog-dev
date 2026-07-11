---
title: "Making .env Secure"
date: 2026-07-07T07:49:14+01:00
draft: false
featured: true
tags: [.env, good engineering, security]
---
*Stop shipping plaintext secrets. Encrypt your `.env`, commit it safely, and let one key unlock it at runtime.*

As developers, we've largely accepted .env files as the standard way to manage environment variables and secrets. Almost every framework, language, and tutorial recommends them.

Yet they're still one of the easiest places for secrets to leak. It's surprisingly common to find API tokens, database passwords, and service credentials stored in plaintext, even in large enterprise solutions. We invest heavily in securing applications and infrastructure, but often overlook the very secrets that unlock them.

Why isn't there more discussion around making .env files secure by default? If they're the de facto standard, shouldn't protecting them be part of the standard too? 🤔🔐

---

## The problem with `.env`

A plain `.env` file is the default way apps read secrets — and it's a liability:

- It sits on disk in **plaintext**, readable by any process running as you.
- It gets **accidentally committed** to git. One `git add .` and your database password is in history forever.
- It has **no rotation, no auditing, no access control**.

> [!CAUTION]
> The single most common secret leak is a `.env` file pushed to a public repo. Search engines and bots scan for them within minutes.

## The fix: encrypt the values, keep the file

[**dotenvx**](https://dotenvx.com) encrypts the *values* in your `.env` while leaving the *keys* readable. The encrypted file is safe to commit. A separate private key — which you never commit — decrypts it at runtime.

```dotenv
DOTENV_PUBLIC_KEY="0339..."
DB_PASSWORD="encrypted:BFt9...=="   # ciphertext, safe in git
```

> [!NOTE]
> Encryption uses a **public key** (in `.env`), decryption uses a **private key** (in `.env.keys` - can be anywhere on your file system). Anyone can *add* a secret; only holders of the private key can *read* one.

## Why this matters

- ✅ **Commit `.env` safely** — it's ciphertext, so it lives in git like any other file.
- ✅ **One secret to inject** — production needs only the private key, not a pile of individual secrets.
- ✅ **Same code everywhere** — your app just reads environment variables; dev, CI, and prod are identical.
- ✅ **Language-agnostic** — dotenvx is a CLI, so C#, Node.js, and Python all use the exact same commands.

## Install (defaults per OS)

| OS | Command |
|----|---------|
| **macOS** | `brew install dotenvx/brew/dotenvx` |
| **Windows** | `winget install dotenvx` |
| **Linux** | `curl -sfS https://dotenvx.sh \| sh` |
| **Any (via npm)** | `npm install -g @dotenvx/dotenvx` |

Verify with `dotenvx --version`.

## The four commands you need

```bash
dotenvx encrypt -f .env                       # encrypt values in place (creates .env.keys)
dotenvx set API_KEY "sk-123" -f .env          # add or update a secret (encrypted)
dotenvx get -f .env                           # view decrypted values (needs .env.keys)
dotenvx run -f .env -- <your app>             # decrypt into env + run your app
```

That's the whole workflow. Write secrets, `encrypt`, then `run`.

> [!TIP]
> `dotenvx set` only needs the **public** key, so teammates can add secrets to a repo without ever seeing existing ones.

## Run it — least code, three languages

Write your `.env`, encrypt it, then run any of these. The app code is one line each because **dotenvx does the work outside the app**.

**C# — [file-based app](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/file-based-programs#create-a-file-based-app)** (single `.cs` file, no project, .NET 10+):

```csharp
// app.cs
#!/usr/bin/env -S dotnet --
Console.WriteLine(Environment.GetEnvironmentVariable("DB_PASSWORD"));
```
```bash
dotenvx run -f .env -- dotnet app.cs
```

> [!TIP]
> The first line is a Unix shebang. Add it, run `chmod +x app.cs`, and you can execute the file directly — **no `dotnet` typed on the command line**:
> ```bash
> dotenvx run -f .env -- ./app.cs
> ```
> On Windows the shebang is ignored; use `dotnet app.cs`.

**Node.js:**

```javascript
// app.js
console.log(process.env.DB_PASSWORD);
```
```bash
dotenvx run -f .env -- node app.js
```

**Python:**

```python
# app.py
import os
print(os.environ["DB_PASSWORD"])
```
```bash
dotenvx run -f .env -- python3 app.py
```

> [!IMPORTANT]
> Notice none of the code imports a secrets library or decrypts anything. dotenvx injects the decrypted values into the process environment, and each language reads them the normal way. Swap dotenvx for Key Vault later and **the code doesn't change**.

## What you MUST `.gitignore`

```gitignore
# The private decryption key — leaking this defeats the whole point.
.env.keys
```

> [!WARNING]
> Commit `.env` (encrypted). **Never** commit `.env.keys`. Run `git status` and confirm `.env.keys` is not listed before your first push.

## Production: where the private key lives

Locally the private key sits in `.env.keys`. In production you never ship that file — you inject the key as a single secret:

```bash
DOTENV_PRIVATE_KEY="<value>" dotenvx run -f .env -- ./myapp
```

That one value is exactly what belongs in **Azure Key Vault**, **AWS Secrets Manager**, or a **systemd credential**, fetched via managed identity. You collapse "many secrets in a file" into "one injectable key," with the encrypted `.env` safely in git.

## This is one of several solutions

dotenvx is the lightest option, but it isn't the only one.

> [!NOTE]
> [**SOPS**](https://github.com/getsops/sops) (Secrets OPerationS) encrypts whole structured files (YAML/JSON/`.env`) against a KMS — **age**, AWS KMS, GCP KMS, or **Azure Key Vault keys**. Reach for SOPS when you want the *encryption key itself* governed by cloud RBAC and audit logs, or when you're encrypting many config files, not just `.env`. Other alternatives: cloud secret managers fetched at runtime, HashiCorp Vault (dynamic, leased secrets), and platform **managed identity** (no secret at all — the strongest option).

Pick based on your needs:

| You want… | Use |
|-----------|-----|
| Minimal change, app reads env vars | **dotenvx** |
| Encryption key under cloud RBAC + audit | **SOPS + Azure Key Vault** |
| Cloud-agnostic, offline, simple | **SOPS + age** |
| No secret at rest at all | **Managed identity** |

## TL;DR

1. `dotenvx encrypt -f .env` → values become ciphertext, `.env.keys` appears.
2. `.gitignore` the `.env.keys`; commit the encrypted `.env`.
3. `dotenvx run -f .env -- <app>` → your one-line C#, Node, or Python reads plain env vars.
4. In production, inject `DOTENV_PRIVATE_KEY` from Key Vault. Done.