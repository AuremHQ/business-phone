# Guide to Bootloader Unlocking — extracted for Business Phone OS

> ✅ VERIFIED — Extracted from the source Notion page + its linked guides on 2026-11-24.
> ⚠️ PARTIALLY VERIFIED — OnePlus: the official support page never rendered its article body (empty article area, no login wall, no CAPTCHA) even in a live browser read, so its steps come from thecustomrom.com's OnePlus guide (LineageOS install pages + OnePlus token/warranty pages, checked Oct 2026) instead — flagged in that section. 5 brands (Xiaomi MIUI, IQOO, Realme, Oppo, Motorola) have no guide linked in the source at all. Everything else below was read in full from the linked sources.
> Source: https://splashy-rooster-65e.notion.site/Guide-to-Bootloader-Unlocking-1be80783d49a8095b649f6f5143aa1d0

**Why this is in this repo:** bootloader unlocking is step 1 for flashing a custom AOSP-based Business Phone OS / Aurem Guard build onto a phone. Device choice decides everything — Pixel and Nothing unlock in minutes with no wait; Xiaomi needs an old account + a quota slot + a 72-hour wait; Samsung can be impossible on some regional/carrier variants.

## Disclaimer (verbatim from source)

> "Do at your own risk, if anything goes wrong, If have guts then only do it."

Try it on your old or secondary device, not your primary phone.

## Universal — how unlocking works (Android Open Source Project)

- Most Android devices ship with a **locked** bootloader — you cannot flash the bootloader or device partitions.
- Get into the bootloader: `adb reboot bootloader`, or hold Volume Down while booting.
- Unlock: `fastboot flashing unlock`. The device **must** show a warning first; after you confirm, it performs a **factory data reset** (so stolen data cannot be read). The unlock flag persists across reboots.
- The command is denied unless `get_unlock_ability` is `1` — set it in **Settings > System > Developer options > OEM unlocking**. If it is `0`, boot to the home screen and enable OEM unlocking first.
- Lock again: `fastboot flashing lock`. Retail devices ship locked (`get_unlock_ability` = `0`).
- Build-time properties: `ro.oem_unlock_supported` = `1` (unlock supported) or `0`; lock state shown by `androidboot.flash.locked` (`1` = locked, `0` = unlocked; in bootconfig from Android 12).
- Critical partitions (fuses, sensor hub, first-stage bootloader): unlocking them needs **physical interaction** — `fastboot flashing unlock_critical` — and devices must never ship in the `unlock_critical` state.
- Source: https://source.android.com/docs/core/architecture/bootloader/locking_unlocking

## Samsung

Source: https://drfone.wondershare.com/unlock/unlock-bootloader-on-samsung.html

**The rule:** enable **OEM Unlocking** in Developer Options, then confirm in **Download Mode**. That confirmation is what unlocks the phone. Expect a **full data wipe**.

**Blocker first:** if the OEM Unlocking toggle is missing or greyed out, your regional/carrier variant does not allow unlocking. No generic PC tool or ADB command bypasses this.

**Steps (Method 1 — on-device):**
1. Connect to Wi-Fi, charge the phone, install any pending official update.
2. Enable Developer Options: Settings > About phone > Software information > tap **Build number 7 times** > back > Developer options > turn **OEM Unlocking** on and confirm the warning.
3. Power off. Hold **Volume Up + Volume Down** together and connect the phone to a PC via USB — this boots Download Mode (exact key combo can vary by model; follow the on-screen instructions).
4. On the "Unlock Bootloader" screen press **Volume Up**, then press and hold **Volume Up** again to confirm.
5. Wait for the wipe + reboot, then set the phone up again and verify backups before flashing anything.

**Steps (Method 2 — with ADB):** enable USB debugging in Developer Options; install ADB; `adb devices` to verify; `adb reboot bootloader` (or the hardware key combo above); follow the Download Mode prompts on screen. `fastboot flashing unlock` only applies if the device actually supports fastboot — on Samsung, use the Download Mode path instead.

**Prerequisites:** back up everything (photos, messages, authenticator codes, work profiles); charge to at least 60%; use a reliable USB cable.

**Costs:** warranty/service eligibility may be affected (depends on region); some security-dependent Samsung services may behave differently; wrong files or steps can cause boot loops or a soft-brick.

**FAQs:** no, you cannot unlock without losing data in most cases; rebooting into Download Mode alone does not unlock anything; if FRP asks for the Google account after the reset, sign in with the last synced account or use Google account recovery — avoid untrusted "bypass" offers.

## Xiaomi HyperOS (Global devices only — NOT China devices)

Sources: https://xiaomi.eu/community/threads/unlock_hyperos_-global-devices.71752/ and https://xdaforums.com/t/how-to-unlock-bootloader-on-xiaomi-hyperos-all-devices-except-cn.4654009/

**Requirements:** a **Global** device (not a China device), a SIM inserted, a Xiaomi account **older than 30 days**, and you can unlock only **1 device per year** per account.

**Step 1 — get permission (Xiaomi Community app):**
1. Install the Xiaomi Community app, change its region to **Global**, log in.
2. Tap **Unlock bootloader** > **Apply for unlocking**.
3. If the quota is reached, wait until it resets at **00:00 China time** and apply again.

**Step 2 — bind the account (after permission is granted):**
1. Log out of the Mi account on the phone, turn off Wi-Fi, restart the phone, turn on **mobile data**.
2. Settings > About phone > tap **HyperOS version** repeatedly until "You are now a developer!"
3. Settings > Additional settings > Developer options: enable **OEM unlocking** and **USB debugging**.
4. Tap **Mi Unlock status** > Agree > **Add account and device** (on mobile data). You should see "Added successfully. Mi account is associated with this device now."
5. A countdown starts — usually **72 hours**. **Do not touch the Mi account on the device during the wait** — re-adding it restarts the timer from zero.

**Step 3 — unlock (after the countdown):**
1. Run the Mi Unlock app on a **Windows 64-bit PC** and sign in with the same Mi account.
2. Power the phone off, hold **Volume Down + Power** to enter Fastboot mode, connect via USB, click **Unlock**. All user data is deleted.

**Quota workaround (XDA script method, when the app quota keeps failing):** Python 3.10+ with Tkinter, plus Firefox/Chrome/Edge; `py -3 -m pip install selenium ntplib urllib3 browser-cookie3`; run `XiaomiUnlock.py`; sign in (new browser window, or reuse a saved Firefox login); schedule the request for **00:00 Beijing time** (max 4 attempts in a 1-second window; keep the PC awake and connected; no automatic next-day retry). After the requests: sign out of the Mi account on the phone, restart, log back in, re-enable Find My Device, re-link the account in Mi Unlock Status, enable OEM unlock, then check the remaining time in MiUnlock. If banned for more than 4 days, add one day. Unlocking erases data.

**China devices:** this guide does not cover them — they need a different thread or a paid service.

## Xiaomi MIUI

- No guide linked in the source Notion page. (Section kept so the gap is visible — add the MIUI-era Mi Unlock flow here when a source is picked.)

## OnePlus

- Official support page (https://service.oneplus.com/us/search/search-detail?id=2085382) never rendered its article body — not even in a live browser read (empty article area; no login wall, no CAPTCHA). OnePlus does not publish a retail unlock guide of its own. The steps below come from thecustomrom.com's OnePlus guide (checked against LineageOS install pages + OnePlus's token/warranty pages, Oct 2026) and itechfaqs — see the source list at the bottom.

**The short version:** global/Indian OnePlus phones unlock with plain fastboot — no account, no application, no waiting. Enable OEM unlocking, reboot to fastboot, run the unlock command, confirm on the phone. Unlocking erases everything.

**Steps:**
1. Settings > About device > Version > tap **Build number** repeatedly until Developer options are enabled.
2. Settings > Developer options: enable **OEM unlocking** and **USB debugging** (menu names can vary between OxygenOS and ColorOS).
3. Install Google's Android SDK Platform-Tools (latest release). `adb devices` — the serial must show as `device`.
4. `adb reboot bootloader` (or the hardware key combo), then `fastboot devices` to confirm the connection.
5. Run the unlock command for your model (see below), confirm on the phone with the volume/power keys. The phone factory-resets.
6. Check state with `fastboot getvar unlocked` or `fastboot getvar all`. Note: the OEM unlocking toggle alone does not prove the bootloader is unlocked — it only permits the attempt.

**Which command (per LineageOS install pages, checked Oct 2026):**
- `fastboot flashing unlock` — OnePlus 13, 12, 12R, 11, Nord CE4.
- `fastboot oem unlock` (older form) — OnePlus 9, 9 Pro, 8 Pro, 8T, Nord, 6.
- Check your exact model number first (Settings > About device) — e.g. OnePlus 13 is CPH2649 / CPH2653 / CPH2655.

**Models that need more than a command:**
- **T-Mobile Edition (US):** needs an unlock token from OnePlus (models from the 6T to the 10 Pro 5G, plus several Nord N phones). Unlock the SIM, turn on OEM unlocking, collect the IMEI with `*#06#` plus an unlock code and serial number from fastboot, and submit them. OnePlus says the process may take **1–2 weeks** and notifies by email.
- **Chinese ColorOS models:** follow a separate Deep Testing application process.
- Carrier-branded units (especially US operators) may ship with the OEM unlocking toggle unavailable.

## IQOO

- No guide linked in the source Notion page.

## Google Pixel

Source: https://source.android.com/docs/core/architecture/bootloader/locking_unlocking (same AOSP rules as the Universal section above).

Pixel is the reference device for the AOSP flow: enable **OEM unlocking** in Developer options, `adb reboot bootloader`, `fastboot flashing unlock`, confirm the on-screen warning, the phone wipes and reboots unlocked. No account, no waiting period, no token.

## Realme

- No guide linked in the source Notion page.

## Oppo

- No guide linked in the source Notion page.

## Nothing

### CMF Phone 1

Source: https://xdaforums.com/t/nothing-cmf-phone-1-bootloader-unlock-guide.4680441/

1. Back up — unlocking **erases all data**. Install ADB and Fastboot on the PC.
2. Developer Options: Settings > About phone > tap **Nothing OS version** > tap **Build number 7 times**.
3. Settings > System > Developer options: enable **USB debugging** and **OEM unlocking**.
4. `adb devices` (check the phone is listed) → `adb reboot bootloader` → `fastboot flashing unlock` → confirm on screen → `fastboot reboot`.
5. The **Orange State** warning on boot is normal. Check unlock status in Developer Options afterwards.
6. Note: CMF Phone 1 is MediaTek — custom ROM support is limited (sources are not public); GSIs still work.

### Nothing Phone 1

Source: https://www.xda-developers.com/how-to-bootloader-unlock-root-magisk-nothing-phone-1/

- Nothing allows unlocking. **No waiting period, no token.**
- Steps: Settings > About phone > Software info > tap **Build number 7 times** > System > Developer options > enable **OEM unlocking** and **USB debugging** (PIN may be asked) → plug into PC with latest ADB/Fastboot → `adb devices` (authorise the prompt) → `adb reboot bootloader` (or Power + Volume Down while booting) → `fastboot flashing unlock` → on screen, press Volume Up until it says "unlock the bootloader", then Power → phone unlocks and reboots to the bootloader menu (red warning icon + "unlocked" text) → `fastboot reboot`.
- For low-level firmware partitions afterwards: `fastboot flashing unlock_critical`.
- Root with Magisk: re-enable Developer Options + USB debugging after the wipe; download the full OTA for the installed version; extract the stock boot image; `adb push <file> /sdcard/Download`; in the Magisk app tap Install > Select and Patch a File; `adb pull /sdcard/Download/magisk_patched_[random].img`; `adb reboot bootloader`; `fastboot flash boot /path/to/magisk_patched_boot.img`; `fastboot reboot`.
- After unlocking/rooting, monthly security patches must be sideloaded manually.

### Nothing Phone 2

Source: https://xdaforums.com/t/nothing-phone-2-bootloader-unlocking-rooting-guide.4702114/ — XDA thread "Nothing Phone 2 Bootloader Unlocking, Relocking & Rooting Guide", post #1 by Arghya Biswas, Nov 9, 2024.

Disclaimer (top of the post): "Proceed with caution as your data will be erased. Take backups accordingly!"

**Unlock:**
1. Install ADB and Fastboot / USB drivers properly.
2. Settings > About phone > tap the **NothingOS banner** at the top > tap **build no. 7 times** to enable Developer options.
3. Developer options: turn on **USB debugging** (grant authorization if asked) and **OEM Unlocking**.
4. `adb devices` (check the ID shows) → `adb reboot bootloader`.
5. `fastboot devices` → `fastboot flashing unlock`.
6. On the phone, use the power and volume keys to select the unlock option. Data is formatted and the device reboots — "Your device is successfully unlocked!"

**Relock (data is erased again — back up first):**
1. **Flash the stock ROM first** — mandatory if you were rooted or changed any partitions; locking in that state would brick the device. A clean stock flash is recommended even otherwise.
2. Reboot to bootloader: `fastboot devices` → `fastboot flashing lock`.
3. Select the on-device lock option with the volume/power keys. The device formats and reboots locked.

**Root with Magisk:**
1. Unlock the bootloader first. Install Magisk Manager (Normal or Delta).
2. Get the stock `boot.img` for your exact NOS version (extract the `-image-boot.7z` file).
3. Magisk app > Install > patch the stock boot image; move the patched `boot.img` to the ADB folder on the PC.
4. Reboot to bootloader: `fastboot devices` → `fastboot boot <magisk patched-boot.img>` — use `fastboot boot`, NOT `fastboot flash boot` (flashing here breaks OTA updates).
5. The phone boots into a temporary rooted environment — do not unplug the USB cable.
6. Magisk app > Install > **Direct install** → reboot. Rooted.

Notes: the guide does not mention `fastboot flashing unlock_critical` and gives no model/region limits. Downloads in the post: USB drivers, stock ROM images, flashing script, visual guide, Magisk Manager, stock boot.img.

## Motorola

- No guide linked in the source Notion page.

## What this means for device choice (our read)

- **Easiest:** Google Pixel and Nothing Phone 1/2 — standard fastboot flow, no account, no wait, no token.
- **Slowest:** Xiaomi HyperOS Global — 30-day-old account + quota slot at midnight China time + 72-hour wait + 1 device per year.
- **Riskiest:** Samsung — some regional/carrier variants cannot be unlocked at all; check the OEM Unlocking toggle before buying a device for the project.
- **Limited ROM future:** MediaTek phones (e.g. CMF Phone 1) — GSIs work, but full custom ROM support is thin.

## Full source list

- Notion list (this extraction's source): https://splashy-rooster-65e.notion.site/Guide-to-Bootloader-Unlocking-1be80783d49a8095b649f6f5143aa1d0
- Samsung guide: https://drfone.wondershare.com/unlock/unlock-bootloader-on-samsung.html
- Xiaomi HyperOS (XDA script method): https://xdaforums.com/t/how-to-unlock-bootloader-on-xiaomi-hyperos-all-devices-except-cn.4654009/
- Xiaomi HyperOS Global (xiaomi.eu): https://xiaomi.eu/community/threads/unlock_hyperos_-global-devices.71752/
- OnePlus support: https://service.oneplus.com/us/search/search-detail?id=2085382 (page never rendered its article body)
- OnePlus guide (secondary source used): https://thecustomrom.com/guides/unlock-bootloader-oneplus
- OnePlus unlock errors / procedure (secondary source used): https://www.itechfaqs.com/oneplus-device-cannot-be-unlocked-due-to-technical-reason-fixes-causes-and-unsupported-models/
- Google Pixel / AOSP locking + unlocking: https://source.android.com/docs/core/architecture/bootloader/locking_unlocking
- Nothing CMF Phone 1: https://xdaforums.com/t/nothing-cmf-phone-1-bootloader-unlock-guide.4680441/
- Nothing Phone 1: https://www.xda-developers.com/how-to-bootloader-unlock-root-magisk-nothing-phone-1/
- Nothing Phone 2: https://xdaforums.com/t/nothing-phone-2-bootloader-unlocking-rooting-guide.4702114/
