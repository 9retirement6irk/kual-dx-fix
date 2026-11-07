# KUAL fix for Kindle DX / DX Graphite

```text
KUAL fix for Kindle DX / DX Graphite (firmware 2.5.x)
=====================================================

Tested on: Kindle DX Graphite (B009), firmware 2.5.8, jailbreak 0.13.N,
Launchpad installed. Result: KUAL opens.


THE PROBLEM
-----------
Since April 2025, KUAL won't open on the Kindle 2 / DX family. You get
"This title is not signed by a registered developer" or "This device is
not registered as a Test Kindle". There are three separate causes:

1. The 2014 developer certificates (MKK) expired in April 2025.
2. The April 2025 replacement KUAL and certificates are signed with
   SHA-256, and the newer signing tools add extra signature fields
   (signing time, algorithm protection). The DX's old Java rejects both.
3. The Kindle deletes developer.keystore whenever it thinks its
   registration changed (wireless on, registration hacks, sometimes
   just exiting KUAL). When that happens you see "not a Test Kindle".


WHAT THIS PACKAGE DOES
----------------------
- documents/KUAL-KDK-1.0.azw2
    The April 2025 KUAL build (2.7.37), re-signed the way 2010-era tools
    did it: SHA-1 digests, SHA1withRSA, and no extra signature fields.
    The manifest keeps only its main attributes.

- kual-dx-fix/developer.keystore
    A new developer keystore (aliases dktest, ditest, dntest; password
    "password"). Its certificates are valid from 1955 to 2097, so the
    Kindle's clock no longer matters, even if it resets to 1970.

- kual-dx-fix/install_keystore.sh
    Copies the keystore to /var/local/java/keystore/developer.keystore
    and makes it immutable (chattr +i) so the Kindle can't delete it.
    Backs up any existing keystore first and writes install.log.

- kual-dx-fix/restore_keystore.sh
    Undo: unlocks the keystore and restores the backup, or removes the
    fix keystore if there was no backup.

- launchpad/kualfix.ini
    Shift K I  = install and lock the keystore
    Shift K U  = undo

- launchpad/settime.ini (optional)
    Shift T T  = set the clock (UTC). The date is written in the file
    as MMDDhhmmYYYY, so edit it before you use it.


REQUIREMENTS
------------
- The Kindle is jailbroken (jailbreak 0.13.N). On firmware 2.x a
  successful jailbreak install ends with "U006 ... not successful".
  That is normal.
- Launchpad is installed (update_launchpad_0.0.1d_dxg_install.bin from
  lpad-pkg-001d.zip; use the dx / dxi file for B004 / B005).
  To check it's running, press Shift three times; a screenshot.bmp
  should appear on the drive.


INSTALL
-------
1. Turn wireless OFF (Menu > Turn Wireless Off) and keep it off.
2. Connect the Kindle to your computer and copy:
     documents/KUAL-KDK-1.0.azw2  ->  documents/ on the Kindle
     kual-dx-fix/                 ->  root of the Kindle drive
     launchpad/*.ini              ->  launchpad/ on the Kindle
   Delete any other KUAL*.azw2 files in documents.
3. Eject the Kindle and restart it (Menu > Settings > Menu > Restart)
   so Launchpad loads the new shortcuts.
4. Press Shift, then K, then I (one after another, not together).
5. Check kual-dx-fix/install.log on the drive. It should say:
     installed keystore OK
     locked (immutable) OK
6. Open KUAL from the Home screen. If it still complains, restart the
   Kindle once and try again.


TROUBLESHOOTING
---------------
"This title is not signed by a registered developer"
  - The KUAL in documents isn't this package's copy, or there are
    several copies. Keep just the one from this package.
  - Or the keystore isn't installed: press Shift K I and check
    install.log.

"This device is not registered as a Test Kindle"
  - The keystore is missing. Press Shift K I, then restart.
  - If it keeps happening, also fake-register the Kindle:
    update_reginfo_0.08 (Update_reginfo_0.08_dxg_install.bin plus
    reginfo.ini, both on the root of the drive), then Update Your Kindle.
    It worked if reginfo_install.log says "Create registration file".

Launchpad shortcuts do nothing
  - Launchpad isn't running. Reinstall it (see Requirements).
  - Make sure the files end in .ini and not .ini.txt.


UNDO
----
Press Shift K U, then restart. To remove everything, delete
documents/KUAL-KDK-1.0.azw2, the kual-dx-fix folder, and
launchpad/kualfix.ini.


NOTES
-----
- The keystore and its private keys are in this package and the
  password is "password". Anyone with this package can sign Kindlets
  your Kindle will trust. That's the same model as the original MKK,
  but keep it in mind before sharing it.
- With the keystore locked, an official developer-certificate update
  can't replace it. Run Shift K U first if you ever need that.
- Keep wireless off. Contacting Amazon can trigger a registration
  change.
- If you have other Kindlets signed with the 2014 MKK keys, they won't
  open with this keystore. Re-sign them with this keystore, using SHA-1
  and no signed attributes (openssl cms -sign -binary -noattr -md sha1).

Credits: KUAL by ixtab, NiLuJe, twobob, stepk. Jailbreak, MKK and
Launchpad by the MobileRead community (NiLuJe, ixtab, h1uke, and
others). This package only re-signs KUAL and supplies a new keystore.
```
