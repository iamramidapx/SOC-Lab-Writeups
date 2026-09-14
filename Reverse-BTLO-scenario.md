# 🕵️ Malware Analysis: Deobfuscating a Multi-Layer PowerShell Payload from a Phishing Macro

## 📌 Scenario

- A phishing email slipped past the mail filter with a `.docm` (macro-enabled Word) attachment.
- IR pulled the file into a sandbox, saw it execute something on open, and flagged it as likely containing shellcode or a downloader.
- Goal: extract the macro, then peel back each layer of obfuscation until we find what the payload actually does — and pull out any Indicators of Compromise (IOCs).

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `olevba` | Extracts VBA macro source code from Office documents |
| `grep` | Pulls the Base64 blob out of the macro dump |
| CyberChef | Visual "recipe" tool for chaining decode/decompress operations |

## 🔍 Step-by-Step Analysis

### Step 1 — Extract the macro
```bash
olevba Macroni.docm
```
This dumps the raw VBA code. Inside it, instead of readable script, there's a wall of text starting with `JAB...` — a classic sign of a Base64-encoded PowerShell command (PowerShell's `-EncodedCommand` blobs almost always start with `JAB`, which decodes to `$`).

### Step 2 — Layer 1: Base64 → UTF-16LE
Recipe: **From Base64** → **Decode Text (UTF-16LE)**

Decoding the blob doesn't immediately give clean text — it comes out looking like `$.s.=.N.e.w.-.o.b.j.e.c.t.` with dots between every character. That's the signature of **UTF-16LE** (2 bytes per character) being misread as plain UTF-8. Re-running "Decode Text" as UTF-16LE (1200) reveals the real PowerShell script underneath.

### Step 3 — Layer 2: Gzip Decompression
Inside that script, two .NET calls give it away:
- `IO.Compression.GzipStream` — treats the next blob as a compressed archive
- `[IO.Compression.CompressionMode]::Decompress` — tells it to inflate that archive in memory

Recipe: **From Base64** → **Gunzip** → **Decode Text**

This unpacks a second PowerShell script.

### Step 4 — Layer 3: XOR Obfuscation
The unpacked script assigns a Base64 string to a byte array:
```powershell
[Byte[]]$var_code = [Convert]::FromBase64String('...')
```
...and a few lines later, XORs every byte in it against a fixed key:
```powershell
$var_code[$x] = $var_code[$x] -bxor 35
```

Recipe: **From Base64** → **XOR (Key: 35)** → **Decode Text**

That final decode reveals the payload's command-and-control (C2) configuration.

## 🚩 Indicators of Compromise (IOC)

| Type | Value |
|---|---|
| Delivery vector | Phishing email → macro-enabled `.docm` attachment |
| C2 IP | `176.103.56.89` |
| Obfuscation chain | Base64 → UTF-16LE → Gzip → XOR (key `35`) |

## 📝 Lessons Learned

- Most malware doesn't need exotic cryptography — **layering simple tricks** (Base64 + compression + XOR + string splitting) is usually enough to slip past basic AV, less-experienced analysts, and automated scanners.
- CyberChef is the single most useful tool in this whole workflow — chaining "recipes" turns a multi-step manual decode into a few drag-and-drop operations.
- Every layer you unwrap tends to reveal another layer. Don't assume you're at the final payload just because the output looks like a script — check what *that* script does before calling it done.
