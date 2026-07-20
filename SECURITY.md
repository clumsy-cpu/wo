# Security Policy

## Transparency, Not Trust

This repo is open source so you can **read the code before you run `pkg install`**. Every package's full contents are here — nothing hidden in binary blobs.

**What to check before installing:**

1. `DEBIAN/postinst` — runs on install. Read it.
2. `DEBIAN/prerm` — runs on uninstall.
3. Files under `data/data/com.termux/files/usr/` — the actual scripts/binaries.

Don't understand the postinst? Copy it, ask an AI "what does this do and is it safe?" Don't install blindly.

## What TermuxVoid Packages **Never** Do

- No package modifies `$PATH`, `$HOME`, `$PREFIX`, or any Termux env var.
- No package touches your existing Termux config.
- We use **symlinks** instead of env mutations — uninstall leaves zero trace.
