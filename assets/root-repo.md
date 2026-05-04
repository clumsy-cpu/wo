# Termux Void Root‑Repo Packages

## 📋 Overview
This repository provides **root‑required** security and penetration testing tools for Termux users on rooted devices. Packages are hosted in the `root` component of the TermuxVoid repository.

> [!NOTE]
> All tools in this repo **require root access** (su/ Magisk) to function properly.  
> Add the root component to your sources list as shown below.

## 📦 Setup

Add the `root` component to your `termuxvoid.list`:

```diff
-deb [trusted=yes arch=all] https://termuxvoid.github.io/repo termuxvoid main
+deb [trusted=yes arch=all] https://termuxvoid.github.io/repo termuxvoid main root
```

Then install the root‑repo metadata package:

```bash
apt update
apt install root-repo
```

After that you can install any root package, e.g.:

```bash
apt install wipwn
```

## 🛠️ Available Root Packages

### 📡 Wi‑Fi & Network
| Tool | Description |
|------|-------------|
| [wipwn](https://github.com/anbuinfosec/wipwn) | Fast automated WiFi WPS PIN cracking tool (requires root for monitor mode) |

> More root packages will be added over time. Stay tuned.

---

> [!WARNING]
> Root access can permanently damage your device or void its warranty.
> These tools are intended for authorised security assessments and educational use only. Misuse against networks you do not own or have explicit permission to test is illegal.

## 🔄 Updates & Support

- Telegram: @nullxvoid
- YouTube: @alienkrishnorg

## ⚠️ Disclaimer

For educational and security research purposes only. Use at your own risk.

