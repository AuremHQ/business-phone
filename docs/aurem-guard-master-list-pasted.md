# Aurem Guard: Consolidated Master List

> Source: pasted by Teji in chat, Tue Nov 24, 2026, 10:57 EST. Verbatim copy for traceability — the consolidated blueprint at `~/workspace/your_files/aurem-guard-final-blueprint.md` folds this in with the other sources.

> **Status:** Sirf concept. Kuch bana ya test nahi hua hai. ORA Dot, harness aur Deploy is list mein shamil nahi hain.

## 1. Goal aur positioning

- Ek secure phone jo **Android par**, **open-weights gatekeeper model** ke saath chale.
- **"Sabse secure" ka daava nahi.** Pitch: "business ke liye samajhne-layak aur jaanchne-layak secure phone."
- Hardened Android/Pixel ke upar ek extra layer: gatekeeper, plain-language rules, audit log.
- iOS par ye layer possible nahi. Android hi pehla platform hai.

## 2. Core design (4 layers)

| Layer | Kya karta hai | Rule |
|---|---|---|
| **Vault** | Data encrypted (AES-256-GCM jaisa), key hardware keystore (StrongBox/TEE) mein | Phone lock par vault band |
| **Policy engine** | Allow / block / ask ka aakhri faisla fixed rules se | Model sirf salah deta hai |
| **Gatekeeper model** | Classify, PII mask, rules translate, local Q&A | Raw key ya poora vault kabhi nahi milta |
| **Choke point** | Saara traffic local VPN se guzarta hai | Pehle cheap rules, model sirf mushkil cases par |
| **Audit log** | Har allow, block, mask, send ka record | User aur admin dono padh sakein |

Aakhri faisla policy engine leta hai, kyunki model prompt injection se behka sakta hai.

## 3. Model

- **Shuruat:** Qwen2.5-0.5B-Instruct (Apache 2.0, ~0.46 GiB Q4_K_M).
- **Alternatives:** SmolLM2-360M, SmolLM2-1.7B (dono Apache 2.0).
- **Kabhi nahi:** Qwen2.5-3B (research-only licence).
- **Install rule:** sirf (1) weights openly downloadable, (2) licence commercial, (3) hash aur signature verify hone ke baad.
- **Model ko user data par train mat karo.** Agar karna pade to sirf DP-SGD ke saath, aur extraction-risk test pehle.
- Runtime licences (llama.cpp, MLC LLM) build time par dobara verify karo.

## 4. Data, keys aur recovery

- **User data weights mein nahi jaayega.** Weights copy ho sakte hain, sawaal se data nikala ja sakta hai, aur exact data wapas nahi milta.
- **Key ko shares mein todo** (Shamir secret sharing, jaise 3 mein se 2).
  - Share 1: phone ka hardware keystore.
  - Share 2: user ka recovery phrase ya dusra device.
  - Share 3: encrypted, akela kuch na khole, ya bilkul nahi.
- **Aurem ke paas user ka data ya key share nahi hoga.** Warna "local means local" ka vaada toot jaata hai.
- Phone lock ya chori par key memory se hat jaati hai. Remote wipe ke baad key delete ho to data bekaar.

## 5. Hashes aur digital prints

- **Hash data wapas nahi deta.** Sirf verify karta hai. Chhote data (phone number, PIN) ka hash brute-force ho sakta hai.
- **Hash kahan rakhna hai:**
  - Integrity check (recovery ke baad data sahi hai ya nahi): phone ke andar.
  - Audit log hash chain (tamper-evident): **sabse zaroori**, ise hatana risky hai.
  - Model signature verify: zaroori.
- **Hash server par upload nahi hoga.**
- **Hash hatane ka option:** sirf tab, jab koi aur check usi kaam ko kare. Audit log ke liye alternative: append-only log + admin-side signed backup.
- **Internet par pehle se maujood footprint poori tarah nahi mita sakte.** Sirf wahi mitao jo hamare control mein hai.

## 6. Footprint policy (sirf hamare control mein)

1. **Local-only:** vault, hash aur audit log phone ya admin device par.
2. **Zero telemetry:** crash ya usage data nahi bhejna. Agar bheje, to bina personal data ke.
3. **Retention limit:** logs 30 din baad apne aap delete (rule tay karna hai).
4. **Crypto-shredding:** key delete karo, data bekaar. File overwrite se zyada reliable, flash storage par bhi.
5. **Backup opt-in aur encrypted.**
6. **Aurem server par user ka identifier ya IP store na ho**, jahan tak possible ho.

## 7. Traffic choke point

- Local VPN service se saara traffic guzarta hai (tracker-blocker jaisa).
- Pehle domain blocklist, model sirf mushkil cases par (battery bachane ke liye).
- Outgoing: tracker/ad block; unknown destination + contacts/location/photos = block + alert; PII mask ya pehle poochho.
- Incoming: phishing link neutralise, malware-like attachment quarantine, scam call flag.

## 8. Honest limits

- ❌ TLS ke andar ka content nahi padh sakta. MITM certificate sell karna red flag hai.
- ❌ Dusri company ko encryption force nahi kar sakta.
- ❌ Chhota model galti karega. Fail-closed: shak ho to block + poochho.
- ❌ Rooted ya unlocked chori hue phone par koi software nahi bachta.
- ❌ Battery aur CPU ka kharcha. POC mein naapna hai.
- ❌ Internet par pehle se phail chuka data wapas nahi aata.

## 9. Use-cases (Secure Phone ke saath)

- Egress allowlist: model real-time mein decide karta hai.
- Install gateway: model file bhi signed gate se aaye.
- Audit log: same log, same record.
- Admin console: kya block hua, kya gaya, kyun, bina user ka private content dekhe.

## 10. Phases aur proof

| Phase | Samay | Kya saabit karna hai |
|---|---|---|
| **0. POC** | 2–3 hafte | Model load, vault khule/band, ek tracker block |
| **1. MVP** | 6–8 hafte | Asli apps chalein, trackers block, battery cost naapa |
| **2. Smart layer** | 6–8 hafte | Local Q&A, plain-language rules, phishing scan |
| **3. Secure Phone integration** | 8–12 hafte | Signed updates, admin console (sirf paying client ke baad) |

- Bechne-layak product tak: chhoti team ke saath 6–9 mahine.

## 11. Tests jo ye saabit karein

- Ek share se key ya data na khule; do shares se wapas mile.
- Unsigned ya tampered model load na ho.
- Model ke kehne par bhi policy engine galat cheez bahar na bheje.
- Phone lock par vault band.
- Audit log mein badlaav pakda jaaye (agar hash chain rakhi).
- Retention ke baad log delete ho.
- Zero telemetry: network capture mein analytics call nahi.

## 12. Bharosa kaise banega

- Threat model likho: kis se bachata hai, kis se nahi.
- Bahari security audit, kisi client ko bechne se pehle.
- Ye sab docs ko "CONCEPT" label ke saath rakho, jab tak POC na ho.

## 13. Open decisions (Teji ke)

1. Android-first, iOS nahi?
2. Qwen2.5-0.5B se shuru karein?
3. POC self-funded ya Secure Phone client ka wait?
4. Naam "Aurem Guard" rakhein ya badlein?
5. Audit log ki hash chain rakhni hai ya alternative?
6. Log retention kitne din (jaise 30)?
7. Secure Phone future-projects mein item #9b ya alag entry?

## 14. Pehla kaam

**POC:** Qwen2.5-0.5B ek Android demo app mein, encrypted vault ke 2 data types, ek tracker domain block. Sawaal ka jawab: kya model phone par itna sahi aur halka chalta hai ki roz use ho sake? Agar nahi, to baaki plan badalna padega.
