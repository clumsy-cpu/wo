## Prerequisites

Before using TermuxVoid, ensure your environment meets these requirements:

- **Termux** installed from [F-Droid](https://f-droid.org/en/packages/com.termux/) (recommended) or GitHub
- **Android 7+** with ~2GB free storage for larger tools
- **Working internet connection** for package downloads
- **No root required** for most tools (some may need root for certain features)

---

## 🔍 Project Overview

This is an **unofficial custom APT repository** for packages not available in the official Termux repositories, specifically compiled and optimized for Android architecture.

> This repository contains tools that are often excluded from official sources due to complexity. All packages are compiled natively for Termux.<br>

## 🔍 Why Open Source?

So you see exactly what runs on your device before it runs. Every package is built from source — nothing is hidden. No TermuxVoid package touches `$PATH`, `$HOME`, or any Termux environment variable. We use **symlinks** instead of env mutations; uninstall leaves zero trace. Don't trust — verify.
