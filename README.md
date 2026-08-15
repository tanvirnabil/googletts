# googletts.agi

Google Text-to-Speech AGI script for Asterisk. Converts text to speech and plays it over a SIP call.

---

## ⚠️ Common Mistakes (Read Before Installing)

Most guides miss two critical steps that will cause your install to fail silently:

1. **Missing `sox` dependency** — the script will not run without it
2. **No volume fix** — Google TTS audio plays at near-zero volume by default. You must patch the script after downloading or you won't hear anything even though it appears to be working.

---

## Installation Guide

### Step 1 — Install Dependencies

```bash
cd /var/lib/asterisk/agi-bin/
sudo apt install libwww-perl libio-socket-ssl-perl liblwp-protocol-https-perl mpg123 sox git -y
```

> `sox` is required for audio conversion and volume control. Most guides forget this.

---

### Step 2 — Download the Script

```bash
cd /var/lib/asterisk/agi-bin/
wget https://raw.githubusercontent.com/tanvirnabil/googletts/main/googletts.agi
```

---

### Step 3 — Fix the Volume Issue (CRITICAL)

By default, Google TTS audio is almost completely silent. You must add a volume gain line to the script.

Open the script and find the `get_sox_args` function (around line 296):

```perl
my @soxargs = ($sox, $source_file, "-q", "-r", $samplerate, "-t", "raw", $dest_file);
if ($speed != 1) {
    ...
}
return @soxargs;
```

Add the gain line **before** the `return` statement:

```perl
my @soxargs = ($sox, $source_file, "-q", "-r", $samplerate, "-t", "raw", $dest_file);
if ($speed != 1) {
    ...
}
push(@soxargs, ("gain", "20"));   # <-- ADD THIS LINE
return @soxargs;
```

Or apply it automatically with sed:

```bash
sed -i 's/^\treturn @soxargs;/\tpush(@soxargs, ("gain", "20"));\n\treturn @soxargs;/' /var/lib/asterisk/agi-bin/googletts.agi
```

Verify the fix was applied:

```bash
grep -n "gain" /var/lib/asterisk/agi-bin/googletts.agi
```

---

### Step 4 — Set Permissions

```bash
chown asterisk:asterisk /var/lib/asterisk/agi-bin/googletts.agi
chmod +x /var/lib/asterisk/agi-bin/googletts.agi
```

---

## Test in Asterisk Dialplan

Edit `/etc/asterisk/extensions_custom.conf` and add:

```
[from-internal-custom]
exten => 1235,1,Answer()
exten => 1235,n,agi(googletts.agi,"This is a simple Google Text-to-Speech test in English.",en)
exten => 1235,n,hangup
```

Reload the dialplan:

```bash
asterisk -rx "dialplan reload"
```

Dial extension **1235** from any SIP phone. You should hear the TTS clearly.

---

## Reusing on Another Server

If you already have a working server with the volume fix applied, the easiest way is to copy the patched script directly:

```bash
scp /var/lib/asterisk/agi-bin/googletts.agi root@YOUR_NEW_SERVER:/var/lib/asterisk/agi-bin/
```

Then on the new server just run Steps 1 and 4. No need to patch again — the fix is already baked into the copied file.
