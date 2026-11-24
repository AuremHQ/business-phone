# Aurem Guard — On-Device Model Vault for the Secure Phone

> ⚠️ STATUS: CONCEPT ONLY — idea stage. Nothing built, nothing tested, no device, no client. This doc develops Teji's Nov 24 night idea into a buildable shape and says plainly where the idea needs correcting.

## The idea in one line

**A small open-source model lives ON the phone, inside the Secure Phone project — every bit of the user's data and every message in or out passes through that model's layer, the model guards it from hacking and tracking, and everything that leaves the phone leaves encrypted.**

- 💡 **Working name:** Aurem Guard (final name Teji's call)
- 📱 **Where it lives:** Secure Phone / Business Phone OS (future-projects item #9) — Android first; iOS cannot do this layer
- 🧠 **The model:** a small **open-weights** model, **0.5B–1.7B parameters**, running fully on-device, no cloud
- 🔒 **The promise:** the user's data never leaves the phone in raw form — the model checks, cleans, and encrypts first

## Why OPEN WEIGHTS — hard rule (Teji, Nov 24 10:17)

- ✅ **Weights khule hon = hum dekh sakte hain model ke andar kya hai.** Koi hidden backdoor nahi, koi chhupa hua data-collection nahi. Ek security product closed black-box model par nahi bikta — customer poochhega "andar kya hai?" to hum weights dikha sakte hain.
- ✅ **Hum model ko apne kaam ke liye badal sakte hain** — security classification / redaction ke liye fine-tune karna, apne rules sikhana. Closed model me ye possible nahi.
- ✅ **Phone par offline chalega, kisi company ke server par nahi** — data kabhi bahar jaata hi nahi model chalane ke liye.
- ✅ **Licence commercial use allow kare** — Apache 2.0 wale models (Qwen 0.5B, SmolLM2) theek hain. Research-only licence (jaise Qwen 3B) ya restrictive licence wale models **banned** hain is product ke liye.
- 📏 **Rule:** model tabhi use hoga jab (1) weights openly downloadable hon, (2) licence commercial ho, (3) weights ka hash verify karke signed install ho.

## The one correction the idea needs (read this first)

- ❌ **User data does NOT go inside the model weights.** Weights are frozen numbers. Stuffing photos, messages, or passwords into them is unreliable (the model cannot fetch them back exactly), and it is a leak — anyone who copies the model file copies the data.
- ✅ **What is real instead:** the model file itself is protected the way weights are protected (signed, encrypted at rest, verified before it loads). The **user data lives in a separate encrypted vault** on the phone. The small model is the **gatekeeper** — nothing reaches the vault and nothing leaves the phone except through the model.
- 💡 **Plain words:** the model is not the safe. The model is the guard standing in front of the safe, checking every person and every parcel in and out.

## How it works — 4 layers

- 🔐 **Layer 1 — The Vault (data at rest):** contacts, photos, messages, files, location history live in an encrypted store on the phone. Keys live in the phone's hardware (StrongBox / TEE) — not in any app, not in the model file. Phone locked = vault locked.
- 🧠 **Layer 2 — The Model Gatekeeper:** the small model reads every request in plain language and decides: allow, block, clean, or ask the user. It is the only thing allowed to open the vault.
- 🌐 **Layer 3 — The Choke Point (data in transit):** all network traffic goes through one local checkpoint on the phone (a local VPN service + proxy, the way tracker-blockers work). Incoming and outgoing both pass through it. The model sees *which app, which address, how much data, what kind* — and blocks trackers, ads, and unknown destinations.
- 📋 **Layer 4 — The Audit Log:** every block, every clean-up, every send is written to a tamper-evident log the user (or the business admin) can read: what left, where it went, and why it was allowed.

## Outgoing flow — what happens when something leaves the phone

- 📤 **App wants to send something → choke point catches it → model checks it:**
- ✅ Tracker / ad / analytics address → **blocked**, logged
- ✅ Unknown new address getting contacts, location, or photos → **blocked + user alert**
- ✅ Text with a phone number, email, or address inside → **model masks it or asks first**
- ✅ Normal message to a known contact → **sent, TLS-encrypted, identifiers stripped**
- 🔑 **Encryption:** TLS for the transport + identifiers removed before sending. True end-to-end encryption only works where the other side supports it — the model cannot force another company's server to encrypt.

## Incoming flow — what happens when something arrives

- 📥 **Message, link, or file arrives → model scans it before the user sees it:**
- ✅ Phishing link / fake bank message → **warning shown, link neutered**
- ✅ Attachment behaving like malware → **quarantined, never opened**
- ✅ Spam / scam call pattern → **flagged**
- 💬 **User asks the phone something about their own data** ("where was I Tuesday?", "find the plumber's number") → the model answers **from the vault, on the phone, nothing uploaded**.

## What the model actually does (its 5 jobs)

- 🛡️ **Classify:** tracker vs legit, safe vs scam, personal vs public
- ✂️ **Redact:** find phone numbers, emails, addresses, locations in outgoing text and mask them
- 🚫 **Block:** stop exfiltration — an app suddenly sending your contacts somewhere new
- 🗣️ **Translate rules:** user says "bank app ko location mat do" → the model turns it into an enforceable rule
- 🔍 **Answer locally:** search and summarise the user's own data without uploading anything

## Model shortlist — verified numbers

| Model | Size | File (quantized) | License | Why it fits |
|---|---|---|---|---|
| **Qwen2.5-0.5B-Instruct** | **494M params** | **~0.46 GiB** (Q4_K_M GGUF) | **Apache 2.0** ✅ VERIFIED | Smallest instruct model, 32K context, commercial use free |
| **SmolLM2-360M-Instruct** | **360M params** | **~0.7 GB** full precision | **Apache 2.0** ✅ VERIFIED | Built for on-device; 1.7B version adds function calling |
| **SmolLM2-1.7B-Instruct** | **1.7B params** | **~1–2 GB** quantized | **Apache 2.0** ✅ VERIFIED | Strongest of the three; needs a better phone |

- ✅ VERIFIED — Qwen2.5-0.5B-Instruct is Apache 2.0, 494M parameters, GGUF Q4_K_M file 491,400,032 bytes (**0.46 GiB**), 32,768-token context.
- ✅ VERIFIED — SmolLM2 family is 135M / 360M / 1.7B, Apache 2.0, "lightweight enough to run on-device".
- ✅ VERIFIED — SmolLM2-1.7B-Instruct supports function calling, text rewriting, and summarisation; the 135M shrinks to about **138 MB** at 8-bit.
- ⚠️ **Trap to avoid:** Qwen2.5-**3B**-Instruct is research-only, non-commercial — do NOT pick the 3B size. The 0.5B, 1.5B, 7B, 14B and 32B are Apache 2.0.
- ⚠️ PARTIALLY VERIFIED — on-device runtimes: **llama.cpp** and **MLC LLM** are the standard open-source phone runtimes (MIT / Apache 2.0). Not re-checked live today — re-verify licences at build time.
- 💡 **Recommendation:** start with **Qwen2.5-0.5B-Instruct** — smallest file, free licence, good enough for classify / redact / block. Move to SmolLM2-1.7B only if the 0.5B proves too weak in testing.

## What it CANNOT do — honest limits

- ❌ **Cannot read inside TLS.** Most app traffic is encrypted end-to-end to its own server. The model sees the destination and the size, not the content. Anyone promising "the model reads all your encrypted traffic" is selling a MITM certificate — that breaks apps and is a red flag, not a feature.
- ❌ **Cannot force other companies to encrypt.** WhatsApp / Signal are E2E by themselves; a random website is not. The model can refuse to send, but it cannot invent encryption on the other side.
- ❌ **A small model can be wrong.** It will sometimes block something legit or miss something sneaky. Design rule: **fail closed** (when unsure, block + ask) + a strict allowlist for business apps.
- ❌ **A rooted or stolen unlocked phone breaks everything.** Hardware keys + vault help, but no software survives a fully compromised phone. Say this to the client plainly.
- ❌ **Battery + speed budget:** a 0.5B model checking every packet costs CPU. The choke point must use cheap rules first (domain lists) and wake the model only for the hard cases.
- ❌ **iOS cannot do this layer.** Apple does not allow a local VPN + proxy to inspect other apps' traffic this way. iOS stays MDM + supervision only (as in the Secure Phone doc).

## Where it plugs into the Secure Phone project

- ✅ **Egress allowlist (already in the Secure Phone blueprint):** the model becomes the brain of the allowlist — instead of a fixed list, it decides in real time.
- ✅ **Install gateway + QR gate:** the model file itself is installed through the same signed gate — no unsigned model can load.
- ✅ **Tamper-evident log:** the model's allow/block log feeds the same log.
- ✅ **Management console:** the business admin sees per-phone: what was blocked, what left, and why — without seeing the user's private content.

## Build plan — phases

- 🚀 **Phase 0 — POC (2–3 weeks):** Qwen2.5-0.5B running in a demo Android app via llama.cpp; encrypted vault with 2–3 data types; one policy (block a test tracker domain). **Prove:** model loads, answers, blocks.
- 🚀 **Phase 1 — MVP (6–8 weeks):** local VPN choke point + tracker blocking + PII redaction + audit log, on one Pixel phone. **Prove:** real apps work, trackers blocked, battery cost measured.
- 🚀 **Phase 2 — Smart layer (6–8 weeks):** on-device Q&A over the vault + plain-language rules ("bank app ko location mat do") + phishing scan on incoming messages.
- 🚀 **Phase 3 — Secure Phone integration (8–12 weeks):** AOSP build with the gates, signed model updates, management console. **Only after a paying client + paid discovery** (same rule as the Secure Phone doc).
- ⚠️ **Total to a sellable product: ~6–9 months with a small team** — not a weekend build. The POC is the cheap proof (2–3 weeks, one developer).

## Decisions needed from Teji

- ❓ **Android-first?** (iOS cannot do this layer — Android is the only real path)
- ❓ **Start with Qwen2.5-0.5B?** (smallest, free licence — my recommendation)
- ❓ **Self-funded POC or wait for the Secure Phone client?** (POC is cheap; the full build needs a paying client, same as the Secure Phone rule)
- ❓ **Working name "Aurem Guard" — keep it or pick another?**
- ❓ **Add this to the future-projects list as item #9b (Secure Phone extension) or its own entry?**

## Sources

- Qwen2.5-0.5B-Instruct — Apache 2.0, 494M params, GGUF 0.46 GiB: https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct
- SmolLM2 family — Apache 2.0, 135M / 360M / 1.7B, on-device: https://huggingface.co/HuggingFaceTB/SmolLM2-360M
- Secure Phone project — future-projects-list.md item #9 (concept + demo only, nothing deployed)
