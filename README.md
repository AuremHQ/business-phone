# Aurem Guard — Final Blueprint

> ⚠️ STATUS: CONCEPT ONLY — sirf concept. Kuch bana ya test nahi hua hai. Ye blueprint Teji ke Nov 24 ke idea, uske pasted Consolidated Master List, aur Secure Phone project ke 5 docs ko ek jagah jodta hai. ORA Dot, harness aur Deploy isme shamil nahi hain.

## 1. Goal aur positioning

- 📱 **Ek secure phone jo Android par, open-weights gatekeeper model ke saath chale.**
- 🚫 **"Sabse secure" ka daava nahi — imaandari se nahi kiya ja sakta.** GrapheneOS (Pixel) aur iPhone Lockdown Mode par saalon ka kaam aur bahari audits lage hain; ek chhoti team unhe OS-level security mein nahi hara sakti.
- ✅ **Pitch:** "business ke liye sabse **samajhne-layak aur jaanchne-layak** secure phone" — hardened Android/Pixel ke upar ek extra layer: gatekeeper model, plain-language rules, audit log.
- ✅ **Strategy:** unke upar khade ho jao, unse lado mat. Pixel ka hardware security chip + hardened Android base ho; Aurem Guard wo layer de jo unke paas nahi hai.
- 📱 **Android-first.** iOS par ye layer possible nahi (Apple local VPN + proxy se dusre apps ka traffic inspect nahi karne deta). iOS track alag hai — ABM + Supervision + MDM + Lockdown Mode (§9).

## 2. Final design — 5 layers

| Layer | Kya karta hai | Rule |
|---|---|---|
| 🔐 **Vault** | Data encrypted (AES-256-GCM jaisa), key hardware keystore (StrongBox / TEE) mein | Phone lock hone par vault band |
| ⚖️ **Policy engine** | Allow / block / ask ka aakhri faisla fixed rules se | Model sirf salah deta hai — aakhri faisla iska |
| 🧠 **Gatekeeper model** | Classify, PII mask, rules translate, local Q&A | Raw key ya poora vault kabhi nahi milta |
| 🌐 **Choke point** | Saara traffic local VPN service se guzarta hai | Pehle cheap rules (domain blocklist), model sirf mushkil cases par |
| 📋 **Audit log** | Har allow, block, mask, send ka tamper-evident record | User aur admin dono padh sakein |

- 💡 **Policy engine aakhri faisla isliye leta hai:** model aane wale messages padhta hai, aur koi message use behka sakta hai (prompt injection). Model ke kehne par bhi policy engine galat cheez bahar nahi bhejta — ye test se saabit karna hai (§10).
- 💡 **Note:** pasted Master List ke heading mein "4 layers" likha tha, table mein 5 hain — sahi ginti **5** hai.

## 3. Decisions — discussion se nikle faisle (locked)

- ✅ **User data weights mein nahi jayega.** Weights copy ho sakte hain, sawaal poochh kar data nikala ja sakta hai, exact data wapas nahi milta, aur model ka size data ke saath nahi badhta. Data encrypted vault mein; model sirf gatekeeper hai.
- ✅ **Open-weights model hi use hoga (hard rule).** Weights khule hon = hum dekh sakte hain andar kya hai, koi hidden backdoor ya chhupa data-collection nahi; hum model ko security kaam ke liye fine-tune kar sakte hain; phone par offline chalega. Closed black-box model par security product nahi bikta.
- ✅ **Model ko user data par train mat karo.** Agar kabhi karna pade to sirf DP-SGD ke saath, aur extraction-risk test pehle.
- ✅ **Key ko shares mein todo, data ko nahi (Shamir secret sharing, 3 mein se 2):** Share 1 = phone ka hardware keystore; Share 2 = user ka recovery phrase ya dusra device; Share 3 = encrypted, akela kuch na khole, ya bilkul nahi.
- ✅ **Aurem ke paas user ka data ya key share nahi hoga.** Warna "local means local" ka vaada toot jaata hai.
- ✅ **Hash sirf jaanch ke liye hai.** Hash data wapas nahi deta — sirf batata hai data badla ya nahi. Chhote data (phone number, PIN) ka hash brute-force ho sakta hai.
- ✅ **Model: Qwen2.5-0.5B-Instruct se shuru karo** (Apache 2.0), signed aur hash-verified install ke saath.
- ✅ **Android-first.** iOS par ye layer mumkin nahi.

## 4. Model — verified numbers

| Model | Size | File (quantized) | License | Role |
|---|---|---|---|---|
| **Qwen2.5-0.5B-Instruct** | **494M params** | **~0.46 GiB** (Q4_K_M GGUF) | **Apache 2.0** ✅ VERIFIED | Shuruat — sabse chhota, free licence, 32K context |
| **SmolLM2-360M-Instruct** | **360M params** | **~0.7 GB** full precision | **Apache 2.0** ✅ VERIFIED | Alternative — on-device ke liye bana |
| **SmolLM2-1.7B-Instruct** | **1.7B params** | **~1–2 GB** quantized | **Apache 2.0** ✅ VERIFIED | Strongest — function calling bhi; behtar phone chahiye |

- ✅ VERIFIED — Qwen2.5-0.5B-Instruct Apache 2.0, 494M parameters, GGUF Q4_K_M file 491,400,032 bytes (**0.46 GiB**), 32,768-token context.
- ✅ VERIFIED — SmolLM2 family 135M / 360M / 1.7B, Apache 2.0, "lightweight enough to run on-device"; 1.7B-Instruct supports function calling, text rewriting, summarisation.
- 🚫 **Kabhi nahi: Qwen2.5-3B-Instruct** — research-only licence, non-commercial.
- 📏 **Install rule (3 shartein, teeno zaroori):** (1) weights openly downloadable hon, (2) licence commercial ho, (3) hash aur signature verify hone ke baad hi install ho.
- ⚠️ PARTIALLY VERIFIED — on-device runtimes **llama.cpp** (MIT) aur **MLC LLM** (Apache 2.0) standard hain; licences build time par dobara verify karni hain.

## 5. Data, keys aur recovery

- 🔐 **Vault:** contacts, photos, messages, files, location history — encrypted store, keys phone ke hardware (StrongBox / TEE) mein. Phone lock = vault lock.
- 🔑 **Recovery (Shamir 3-of-2):** ek share se kuch na khule; do shares se key wapas mile. Phone lock ya chori par key memory se hat jaati hai. Remote wipe ke baad key delete ho to data bekaar (crypto-shredding, §6).
- 🚫 **Aurem ke paas kuch nahi:** na user ka data, na key share. "Local means local" — is vaade par poora product khada hai.
- 📥 **User apne data se sawaal poochhe** ("Tuesday ko main kahan tha?") → model vault se jawab de, phone par, kuch upload kiye bina.

## 6. Hashes aur digital prints — footprint policy

### Hash kahan rakhna hai

| Hash kahan use hota hai | Hatane par kya khota hai | Faisla |
|---|---|---|
| Data integrity check (recovery ke baad verify) | Pata nahi chalega data badla ya corrupt hua | 📱 Phone ke andar rakho |
| Audit log hash chain (tamper-evident) | Log chupke se badla ja sakta hai, proof nahi rahega | 📋 **Sabse zaroori — hatana risky hai** |
| Model file signature/hash verify | Modified model load ho sakta hai, security gate khatam | 🔒 Zaroori — rakho |

- ✅ **Hash hatana tab hi theek hai jab koi aur check usi kaam ko kare.** Audit log ke liye alternative: append-only log + admin-side signed backup. (Teji ka faisla baki hai — §12, sawal 5.)
- 🚫 **Hash server par upload nahi hoga.** Integrity check phone ke andar hi ho.
- 🚫 **Internet par pehle se maujood footprint poori tarah nahi mita sakte.** Jo data server logs, third-party trackers, backups, search cache mein ja chuka hai, wo wahan rehta hai jab tak uski company delete na kare. Sirf wahi mitao jo hamare control mein hai.

### Footprint policy — sirf hamare control mein (7 rules)

1. 🏠 **Local-only by default:** vault, hash aur audit log phone ya admin device par. Koi cloud sync nahi.
2. 📵 **Zero telemetry:** crash reports ya usage analytics na bhejo. Agar bhejo, to bina personal data ke.
3. ⏱️ **Retention limit:** logs expiry ke baad apne aap delete (jaise 30 din — Teji ka faisla baki, §12 sawal 6).
4. 🔥 **Crypto-shredding:** key delete karo, data bekaar. File overwrite se zyada reliable, flash storage par bhi.
5. 💾 **Backups:** user ko opt-in dena, aur backup bhi encrypted ho.
6. 🕵️ **Hamare apne logs:** Aurem ke server par user ka koi identifier ya IP store na ho, jahan tak possible ho.
7. 🚫 **Koi hash upload nahi:** integrity check phone ke andar hi ho, server par hash na jaaye.

## 7. Traffic choke point — aane-jaane wala sab kuch

- 🌐 **Local VPN service** se saara traffic guzarta hai (tracker-blocker jaisa). Pehle domain blocklist (cheap), model sirf mushkil cases par — battery bachane ke liye.
- 📤 **Outgoing:** tracker / ad / analytics address → **block**; unknown destination + contacts/location/photos → **block + alert**; text mein phone number / email / address → **mask ya pehle poochho**; normal message known contact ko → **TLS-encrypted, identifiers stripped**.
- 📥 **Incoming:** phishing link → **warning, link neutralise**; malware-like attachment → **quarantine, kabhi na khole**; scam call pattern → **flag**.
- ❌ **TLS ke andar ka content model nahi padh sakta** — sirf destination aur size. MITM certificate bechna red flag hai, feature nahi.

## 8. Secure Phone ke saath use-cases

- ✅ **Egress allowlist (blueprint mein pehle se hai):** model uska brain ban jata hai — fixed list ki jagah real-time faisla.
- ✅ **Install gateway + QR gate:** model file bhi signed gate se aaye — unsigned model load hi na ho.
- ✅ **Audit log:** wahi log, wahi record — model ke allow/block bhi usi mein.
- ✅ **Admin console:** business admin dekhe — kya block hua, kya gaya, kyun — bina user ka private content dekhe.
- ✅ **Private Line (VoIP second number):** SIM number kabhi na dikhe, no spoofing (Secure Phone docs se).

## 9. Honest limits — jo ye kabhi nahi kar sakta

- ❌ TLS ke andar ka content nahi padh sakta, sirf destination aur size.
- ❌ Dusri company ko encryption force nahi kar sakta.
- ❌ Chhota model galti karega — isliye fail-closed: shak ho to block + user se poochho.
- ❌ Rooted ya unlocked chori hue phone par koi software nahi bachta.
- ❌ Battery aur CPU ka kharcha hoga — kitna, POC mein naapna hai.
- ❌ Internet par pehle se phail chuka data wapas nahi aata.
- ❌ Koi phone "100% safe" nahi — residual risk (user Allow dabaye, social engineering, naye 0-day) seedha bolna hai.

## 10. Phases, proof aur tests

| Phase | Samay | Kya saabit karna hai |
|---|---|---|
| 🚀 **0. POC** | **2–3 hafte** | Model load ho, vault khule/band ho, ek tracker block ho |
| 🚀 **1. MVP** | **6–8 hafte** | Asli apps chalein, trackers block hon, battery cost naapa jaye |
| 🚀 **2. Smart layer** | **6–8 hafte** | Local Q&A, plain-language rules, phishing scan |
| 🚀 **3. Secure Phone integration** | **8–12 hafte** | Signed updates, admin console (sirf paying client ke baad) |

- 📊 **Bechne-layak product tak: chhoti team ke saath 6–9 mahine.** POC self-funded (sasta hai, asli sawaal ka jawab deta hai); Phase 3 sirf paying client ke baad.

### Tests jo ye saabit karein

- ✅ Ek share se key ya data na khule; do shares se wapas mile.
- ✅ Unsigned ya tampered model load na ho.
- ✅ Model ke kehne par bhi policy engine galat cheez bahar na bheje.
- ✅ Phone lock par vault band.
- ✅ Audit log mein badlaav pakda jaaye (agar hash chain rakhi).
- ✅ Retention ke baad log delete ho.
- ✅ Zero telemetry: network capture mein analytics call nahi.

## 11. Bharosa kaise banega

- 📝 **Threat model likho:** kis se bachata hai, kis se nahi — POC se pehle.
- 🔍 **Bahari security audit** karwao, kisi client ko bechne se pehle.
- 🏷️ **Ye sab docs "CONCEPT" label ke saath rakho**, jab tak POC na ho.
- 📚 **Research base (public only):** Project Zero, Citizen Lab, Amnesty Tech. No malware reverse-engineering.

## 12. Open decisions (Teji ke)

1. ❓ Android-first, iOS nahi?
2. ❓ Qwen2.5-0.5B se shuru karein?
3. ❓ POC self-funded ya Secure Phone client ka wait?
4. ❓ Naam "Aurem Guard" rakhein ya badlein?
5. ❓ Audit log ki hash chain rakhni hai ya alternative (append-only + signed backup)?
6. ❓ Log retention kitne din (jaise 30)?
7. ❓ Secure Phone future-projects mein item #9b ya alag entry?

## 13. Pehla kaam

- 🚀 **POC:** Qwen2.5-0.5B ek Android demo app mein, encrypted vault ke 2 data types, ek tracker domain block. Sawaal ka jawab: kya model phone par itna sahi aur itna halka chalta hai ki roz use ho sake? **Agar nahi, to baaki plan badalna padega.**

## 14. Inventory — saari files, skills aur usable cheezein

### Files — Secure Phone docs (Nov 13–18, 2026)

| File | Kya hai | Status |
|---|---|---|
| `~/workspace/your_files/Secure_Phone_Project_Engineering_Documentation_14_6wfh.pdf` | Secure Phone Project Engineering Documentation (Nov 13, 7 pages) | ✅ VERIFIED — file mili |
| `~/workspace/your_files/Secure_Phone_Engineering_Blueprint_16_w5d1.md` | Secure Phone Engineering Blueprint (MD) | ✅ VERIFIED — file mili |
| `~/workspace/your_files/Secure_Phone_Engineering_Blueprint_17_jw4s.html` | Secure Phone Engineering Blueprint (HTML) | ✅ VERIFIED — file mili |
| `~/workspace/your_files/Business_Phone_OS_Architecture__Roadmap_and_Blueprint_18_gxps.pdf` | Business Phone OS Architecture, Roadmap and Blueprint (18 pages) | ✅ VERIFIED — file mili |
| `~/workspace/secure-phone-architecture-map.html` | Architecture map (HTML) | ✅ VERIFIED — file mili, content dobara nahi padha |
| iOS Managed Deployment Blueprint | iOS track ka blueprint | ⚠️ PARTIALLY VERIFIED — docs mein referenced hai, alag file VM par nahi mili |
| `BusinessPhone-Restrictions.mobileconfig` | iOS restrictions profile (2 payloads: Restrictions + Global HTTP Proxy, placeholder host) | ⚠️ PARTIALLY VERIFIED — plistlib se validated (docs ke mutabiq), file VM par nahi mili |
| `InstallGateDemo.zip` (Kotlin, compileSdk 34, minSdk 26) | Install gateway demo | ❌ ROUTE NOT VERIFIED — file VM par nahi mili; kabhi compile nahi hua (no SDK), no tests, no device testing |
| 5 skeleton files (`03_parser_sandbox.py`, `04_egress_allowlist.py`, `05_tamper_evident_log.py` + 2) | Code-level gates ke demo | ⚠️ PARTIALLY VERIFIED — sirf 03 aur 05 chale the, 04 sirf compile hua; files VM par nahi mili |

### Files — Aurem Guard (Nov 24, 2026)

| File | Kya hai | Status |
|---|---|---|
| `~/workspace/your_files/aurem-guard-final-blueprint.md` | **Ye blueprint** — final consolidated doc | ✅ VERIFIED — ye file |
| `~/workspace/your_files/secure-phone-model-vault-idea.md` | Pehla idea doc (Nov 24 10:15) — vault + gatekeeper, model shortlist, limits | ✅ VERIFIED |
| `~/workspace/your_files/aurem-guard-master-list-pasted.md` | Teji ka pasted Consolidated Master List (Nov 24 10:57) — verbatim copy | ✅ VERIFIED |
| `~/workspace/your_files/future-projects-list.md` (item #9) | Secure Phone entry — is blueprint ka pointer add hua | ✅ VERIFIED |

### Skills — jo is kaam mein use hongi

| Skill | Kya karti hai | Aurem Guard mein kaise use hogi |
|---|---|---|
| **skill-security-scan** (`~/workspace/custom-skills/skill-security-scan/SKILL.md`) | SkillSpector v2.12.0 (`~/.local/bin/skillspector`) se kisi bhi skill/model file ko scan karo; critical/high par block | 🔒 **HARD RULE:** model file, skill, ya plugin install/use karne se pehle scan — critical/high mile to install roko jab tak fix ya Teji accept na kare |
| **press-release-formatter** (`~/workspace/custom-skills/press-release-formatter/SKILL.md`) | Teji ke docs ka format — status banner, bold numbers, badges, blunt tone | 📝 Ye blueprint isi format mein hai |
| **documents** (`/opt/muse/skills/documents/SKILL.md`) | Docs, PDF, spreadsheet padhna/likhna/revise karna | 📄 Secure Phone PDFs aur is blueprint ko revise karne ke liye |

### Usable cheezein — models, runtimes, pehle se bana hua

| Cheez | Detail | Status |
|---|---|---|
| **Qwen2.5-0.5B-Instruct** | Apache 2.0, 494M, ~0.46 GiB GGUF, 32K context | ✅ VERIFIED (§4) — POC ka model |
| **SmolLM2-360M / 1.7B-Instruct** | Apache 2.0, on-device design | ✅ VERIFIED (§4) — alternatives |
| **llama.cpp / MLC LLM** | On-device runtimes | ⚠️ PARTIALLY VERIFIED — licences build time par dobara verify karni hain |
| **Secure Phone gates** | Install gateway, QR gate, sandboxed parsers, egress allowlist, tamper-evident log (Gate 0 intake triage, Gate 1 install/permission gateway, parser isolation, sandbox, OS integrity) | ⚠️ PARTIALLY VERIFIED — design docs mein hai, demo kabhi device par nahi chala |
| **iOS track** | Apple Business Manager → Supervision → MDM →.mobileconfig → Lockdown Mode | ⚠️ PARTIALLY VERIFIED — mobileconfig validated, file VM par nahi |
| **Private Line** | VoIP second number, SIM number kabhi na dikhe | 💡 Concept — Secure Phone docs se |
| **Pixel prototype plan** | 8–10 hafte | 💡 Plan — docs se |
| **Admin console** | Remote management, recurring revenue | 💡 Concept — business model |

### Business — Secure Phone docs se

- 👥 **Team ~8–12, ship ~12–18 mahine.** Rough cost **~USD 1.5–3.5M** (approximate planning figures, docs se).
- 💰 **Do NOT build an OS from scratch.** Recurring revenue via remote management console.
- 📋 **Paid discovery phase pehle.** Client ko quote karne se pehle 5 sawaal: hardware/BSP, Play Store/GMS licence, volume/countries, 3–5 saal patches kaun dega, budget.
- 🛡️ **DEFENSIVE security only** — phone ko hacking se bachana, offensive hacking tool nahi.

## Sources

- Teji ka idea (chat, Nov 24 10:15) + open-weights clarification (10:17) + pasted Final Report (10:55) + pasted Consolidated Master List (10:57)
- `~/workspace/your_files/secure-phone-model-vault-idea.md` (Nov 24 10:15)
- Secure Phone docs: `Secure_Phone_Project_Engineering_Documentation_14_6wfh.pdf`, `Secure_Phone_Engineering_Blueprint_16_w5d1.md` + `_17_jw4s.html`, `Business_Phone_OS_Architecture__Roadmap_and_Blueprint_18_gxps.pdf` (Nov 13–18)
- Future projects list item #9 + daily memory note 2026-11-18 (gates, InstallGateDemo, mobileconfig, cost figures)
- Qwen2.5-0.5B-Instruct: https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct
- SmolLM2 family: https://huggingface.co/HuggingFaceTB/SmolLM2-360M
