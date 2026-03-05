# 🔐 Password Audit via GitHub Issues

[![GitHub Actions](https://img.shields.io/badge/Powered%20by-GitHub%20Actions-blue?logo=github-actions)](https://github.com/features/actions)
[![John the Ripper](https://img.shields.io/badge/Engine-John%20the%20Ripper-red)](https://www.openwall.com/john/)
[![Educational](https://img.shields.io/badge/Purpose-Educational%20Only-yellow)](https://github.com/unaveragetech/john-x-ripper)

> **Open an issue, get an instant, comprehensive password security report — powered by John the Ripper and an extensive pattern-analysis engine.**

---

## 🧭 What Is This?

This repository uses a **GitHub Actions workflow** to audit a password's security whenever a new issue is opened with the title `crack`. The workflow automatically:

1. 🔍 **Parses** the password from the issue body (and optional flags)
2. 📊 **Analyses** it with an extensive security-pattern engine (see below)
3. 🔨 **Hashes** it with `openssl passwd` (SHA-512crypt by default)
4. ⚡ **Runs John the Ripper** to attempt to crack it within a time limit
5. 📝 **Posts a detailed report** as a comment (mentioning you), then closes the issue
6. 🔒 **Redacts** the password from the issue body for privacy

The password analysis engine checks for:
- Known common/bad passwords (100+ entries)
- Leet-speak substitutions of dictionary words (`p@ssw0rd`, `@dmin`, etc.)
- Keyboard-walk patterns (`qwerty`, `asdf`, `1q2w3e`, etc.)
- Sequential character runs (`123456`, `abcde`, etc.)
- Repeated characters (`aaaa`, `1111`, etc.)
- Year and date patterns (`1990`, `0101`, etc.)
- "Word + number" and "word + symbol" predictable structures
- Missing character classes (no uppercase, no symbols, etc.)

---

## 🚀 Quick Start

1. **[Open a new issue](../../issues/new)**
2. **Set the title** to exactly: `crack` *(case-sensitive)*
3. **Put your password in the body** (first non-blank, non-flag line):
   ```
   MyPasswordToTest
   ```
4. **Submit** — the workflow starts immediately. Within ~2 minutes you'll receive a comment with a full audit report, then the issue is automatically closed and the password is redacted.

---

## ⚙️ Optional Flags

Add these on separate lines *after* the password:

| Flag | Default | Description |
|---|---|---|
| `--hash=sha512crypt` | `sha512crypt` | Hash algorithm: `sha512crypt`, `sha256crypt`, or `md5crypt` |
| `--time=30` | `30` | Cracking time limit in seconds (10–120) |
| `--mode=auto` | `auto` | Attack mode: `auto`, `wordlist`, `incremental`, `single` |

**Example with all flags:**
```
Tr0ub4dor&3
--hash=sha256crypt
--time=60
--mode=auto
```

### Attack Modes

| Mode | Description |
|---|---|
| `auto` | Tries wordlist first, then incremental brute-force with remaining time |
| `wordlist` | Only runs the built-in word list (`password.lst`) |
| `incremental` | Character-by-character brute-force |
| `single` | Uses login name and GECOS field mutations |

---

## 📊 Understanding the Report

The report produced by the audit contains several sections:

### 📈 Audit Statistics
Basic numbers: password length, character pool size, entropy, hash format used, cracking time, and hardware benchmark speed.

### 🔡 Character Composition
A breakdown of how many lowercase, uppercase, digit, and special characters the password contains.

### 🔢 What Is Entropy? (Plain English)
**Entropy** (measured in bits) tells you how many possible combinations an attacker would need to try to guess your password *if it were chosen randomly from a uniform distribution*.

```
entropy = length × log₂(character pool size)
```

For example, a 10-character password using only lowercase letters has a pool of 26:
```
10 × log₂(26) ≈ 47 bits → ~140 trillion combinations
```

**The critical caveat:** Entropy is theoretical. It assumes the password was chosen *randomly*. If `password1!` appears in a wordlist, an attacker finds it in milliseconds — the entropy formula doesn't know that.

| Grade | Entropy Range | What it means |
|---|---|---|
| 🔴 Very Weak | < 28 bits | Guessed in milliseconds |
| 🟠 Weak | 28–36 bits | Guessed in seconds |
| 🟡 Moderate | 36–60 bits | Minutes to hours on modern hardware |
| 🟢 Strong | 60–128 bits | Years to crack with dedicated hardware |
| 💪 Very Strong | 128+ bits | Effectively uncrackable by brute force |

### 🚨 Security Pattern Analysis
This section lists every insecurity issue detected, each with a **severity level**:
- 🔴 **Critical** — will be cracked almost instantly (e.g., in a common passwords list)
- 🟠 **High** — significantly weakens the password (e.g., dictionary word, keyboard walk)
- 🟡 **Medium** — reduces security (e.g., missing character class, date pattern)
- 🔵 **Low** — minor improvement opportunity

### 💪 Overall Verdict
The **overall grade** is the *worst* of:
- The entropy-based grade
- The pattern-analysis grade

This means `password1!` (entropy grade: Strong) correctly shows as **Very Weak** because it is found in the common passwords list.

### 💡 Password Improvement Tips
Actionable advice on how to create a stronger, more secure password.

---

## 🔒 Security & Privacy

> ⚠️ **IMPORTANT: Never submit a real, currently-used password.**
>
> GitHub issues are **public** by default. Anyone can see the issue body before it is redacted. The password is masked in workflow logs, but it is visible in the issue body for the ~2 minutes the workflow runs.

**What the workflow does to protect your password:**
- Uses `::add-mask::` to hide the password from all subsequent log lines
- Pipes the password through a file rather than expanding it on the command line (prevents shell injection)
- Replaces the issue body with `[Password redacted after audit completed]` after the audit

**Best practices:**
- Test passwords you are *considering* using — not ones already deployed
- Change any real password *before* submitting it here
- Fork this repository and make it **private** if you want fully private audits

---

## ⚡ Performance: Caching John the Ripper

The workflow uses `actions/cache` to cache the John the Ripper binary and word lists between runs. This avoids reinstalling (~30–60 s of `apt-get`) on every audit, making repeat runs significantly faster.

The cache key is `john-<OS>-apt-v1`. If the John the Ripper version needs to be updated, increment the version suffix in the workflow.

---

## 📖 Full Documentation

See [index.html](index.html) (or the [GitHub Pages site](https://unaveragetech.github.io/john-x-ripper/)) for the full usage guide, including:
- All available flags and examples
- Detailed explanations of every report field
- FAQ
- How the workflow works step-by-step

---

## 🛠️ Technical Details

| Component | Details |
|---|---|
| **Workflow trigger** | `issues.opened` where `title == 'crack'` |
| **Runner** | `ubuntu-latest` |
| **Hash tool** | `openssl passwd` (-6 / -5 / -1) |
| **Cracking engine** | John the Ripper 1.9.0 (Ubuntu package) |
| **Analysis engine** | Python 3 (embedded in workflow) |
| **Report posting** | `actions/github-script` v7 |
| **Timeout** | 10 minutes (workflow), 10–120 s (cracking) |

---

## 📜 License

This repository's **workflow and tooling** is released under the MIT License.  
John the Ripper is released under the **GNU GPL v2+** license — see [doc/LICENSE](doc/LICENSE).
