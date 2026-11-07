# KUAL fix for Kindle DX / DX Graphite

Get **KUAL** opening again on the **Kindle DX, DX International and DX Graphite** (firmware 2.5.x), after the 2025 developer-certificate expiry broke it.

![Firmware](https://img.shields.io/badge/firmware-2.5.x-blue)
![Devices](https://img.shields.io/badge/devices-DX%20%7C%20DXi%20%7C%20DXG-lightgrey)
![Tested](https://img.shields.io/badge/tested-DXG%20B009%20%2F%202.5.8-brightgreen)

> **Tested on:** Kindle DX Graphite (B009), firmware 2.5.8, jailbreak 0.13.N, Launchpad 0.0.1d. Result: KUAL opens.

---

## Contents

- [The problem](#the-problem)
- [What's in this repo](#whats-in-this-repo)
- [Requirements](#requirements)
- [Install](#install)
- [Troubleshooting](#troubleshooting)
- [Undo](#undo)
- [Notes and warnings](#notes-and-warnings)
- [Credits](#credits)

---

## The problem

Since **April 2025**, KUAL won't open on the Kindle 2 / DX family. You get one of these errors:

- *"This title is not signed by a registered developer"*
- *"This device is not registered as a Test Kindle to run this title"*

There are three separate causes:

| # | Cause | Effect |
|---|-------|--------|
| 1 | The 2014 developer certificates (MKK) **expired in April 2025**. | Old KUAL builds only open with the clock set back. |
| 2 | The 2025 KUAL and certificates are signed with **SHA-256**, and the newer signing tools add **extra signature fields** (signing time, algorithm protection). | The DX's old Java rejects both, so the official fix doesn't work on these models. |
| 3 | The Kindle **deletes `developer.keystore`** whenever it thinks its registration changed (wireless on, registration hacks, sometimes just exiting KUAL). | "Not a Test Kindle" error, plus a *Test Kindle Installation Result* book in the library. |

This package deals with all three.

---

## What's in this repo

```text
kual-dx-fix/
├── documents/
│   └── KUAL-KDK-1.0.azw2        ← re-signed KUAL 2.7.37
├── kual-dx-fix/
│   ├── developer.keystore       ← non-expiring developer certificates
│   ├── install_keystore.sh      ← installs + locks the keystore
│   └── restore_keystore.sh      ← undo
└── launchpad/
    ├── kualfix.ini              ← Shift K I / Shift K U
    └── settime.ini              ← Shift T T (optional clock setter)
```

| File | What it does |
|------|--------------|
| `KUAL-KDK-1.0.azw2` | The April 2025 KUAL build, re-signed the way 2010-era tools did it: **SHA-1 digests, SHA1withRSA, no extra signature fields**. The manifest keeps only its main attributes. |
| `developer.keystore` | New keystore with aliases `dktest`, `ditest`, `dntest` (password `password`). Its certificates are **valid 1955–2097**, so the clock doesn't matter, even if it resets to 1970. |
| `install_keystore.sh` | Copies the keystore to `/var/local/java/keystore/developer.keystore` and makes it **immutable** (`chattr +i`) so the Kindle can't delete it. Backs up any existing keystore and writes `install.log`. |
| `restore_keystore.sh` | Unlocks the keystore and restores the backup, or removes the fix keystore if there was no backup. |
| `kualfix.ini` | Launchpad shortcuts: **Shift K I** installs, **Shift K U** undoes. |
| `settime.ini` | **Shift T T** sets the clock. The date is written in the file in UTC as `MMDDhhmmYYYY`, so **edit it before you use it**. |

---

## Requirements

- [x] **Jailbroken** with `kindle-jailbreak-0.13.N`
  > On firmware 2.x a successful jailbreak ends with **U006 "update not successful"**. That's normal. Don't keep re-running it.
- [x] **Launchpad** installed: `update_launchpad_0.0.1d_<model>_install.bin` from `lpad-pkg-001d.zip` ([MobileRead thread](https://www.mobileread.com/forums/showthread.php?p=1155117))
  > **Check:** press **Shift** three times. If a `screenshot.bmp` appears on the drive, Launchpad is running.

Pick the right package for your serial number (Menu → Settings):

| Serial prefix | Model | Package suffix |
|---------------|-------|----------------|
| `B004` | Kindle DX (US) | `dx` |
| `B005` | Kindle DX International | `dxi` |
| `B009` | Kindle DX Graphite | `dxg` |

---

## Install

1. **Turn wireless off** (Menu → Turn Wireless Off) and keep it off.
2. Connect the Kindle to your computer and copy:

   | From this repo | To the Kindle |
   |----------------|---------------|
   | `documents/KUAL-KDK-1.0.azw2` | `documents/` |
   | `kual-dx-fix/` (whole folder) | root of the drive |
   | `launchpad/*.ini` | `launchpad/` |

   Delete any **other** `KUAL*.azw2` files in `documents`.
3. Eject the Kindle and **restart** it (Menu → Settings → Menu → Restart) so Launchpad loads the new shortcuts.
4. Press **Shift**, then **K**, then **I**, one after another (not together).
5. Open `kual-dx-fix/install.log` on the drive and check it says:
   ```text
   installed keystore OK
   locked (immutable) OK
   ```
6. **Open KUAL** from the Home screen. If it still complains, restart once and try again.

---

## Troubleshooting

<details>
<summary><b>"This title is not signed by a registered developer"</b></summary>

- The KUAL in `documents` isn't this repo's copy, or there's more than one. Keep only the one from this repo.
- Or the keystore isn't installed. Press **Shift K I** and check `install.log`.
- The Kindle only reads the keystore at startup, so restart after installing it.

</details>

<details>
<summary><b>"This device is not registered as a Test Kindle"</b></summary>

- The keystore is missing. Press **Shift K I**, then restart.
- If it keeps happening, also fake-register the Kindle with **update_reginfo_0.08**:
  copy `Update_reginfo_0.08_<model>_install.bin` **and** `reginfo.ini` to the root of the drive, then run *Update Your Kindle*.
  It worked if `reginfo_install.log` says **"Create registration file"** and Settings shows a registered name.

</details>

<details>
<summary><b>Launchpad shortcuts do nothing</b></summary>

- Launchpad isn't running. Reinstall it (see [Requirements](#requirements)).
- Make sure the files end in `.ini`, not `.ini.txt` (Windows hides extensions).
- Restart after adding or editing `.ini` files.

</details>

<details>
<summary><b>The clock keeps resetting to 1970</b></summary>

The fix doesn't care, because the certificates are valid from 1955. If you want the right time anyway, edit `settime.ini` and press **Shift T T**. Check it by typing `;date` in the Home screen search box.

</details>

---

## Undo

Press **Shift K U**, then restart.

To remove everything, delete:
- `documents/KUAL-KDK-1.0.azw2`
- the `kual-dx-fix/` folder
- `launchpad/kualfix.ini`

---

## Notes and warnings

> [!WARNING]
> **The keystore's private keys are in this repo and the password is `password`.** Anyone can sign Kindlets that a Kindle with this keystore will trust. That's the same model as the original MKK, but keep it in mind.

- With the keystore **locked**, an official developer-certificate update can't replace it. Run **Shift K U** first if you ever need that.
- **Keep wireless off.** Contacting Amazon can trigger a registration change.
- Other Kindlets signed with the **2014 MKK keys** won't open with this keystore. Re-sign them with this keystore, using SHA-1 and no signed attributes:
  ```sh
  openssl cms -sign -binary -noattr -md sha1 -nosmimecap -outform DER \
    -in META-INF/DKTEST.SF -signer dktest.crt -inkey dktest.key -out META-INF/DKTEST.RSA
  ```
- Use at your own risk. Nothing here modifies the root filesystem; the only system change is one file in `/var/local/java/keystore/`.

---

## Credits

- **KUAL** by ixtab, NiLuJe, twobob and stepk
- **Jailbreak, MKK and snapshots** by NiLuJe, ixtab and the MobileRead community
- **Launchpad** by h1uke
- **update_reginfo** by Sir Alex, Nifty and Panopticon
- DX guide and workarounds by vendo232 and SpartanHeavy on [MobileRead](https://www.mobileread.com/forums/showthread.php?p=4475047)

This repo only re-signs KUAL and supplies a new keystore. All credit for the tools goes to their authors.
