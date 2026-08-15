# googletts.agi

Google Text-to-Speech AGI script for Asterisk.

---

## Install Google TTS

### 1. Install dependencies

```bash
sudo apt install libwww-perl libio-socket-ssl-perl liblwp-protocol-https-perl mpg123 git -y
```

### 2. Download from GitHub

```bash
cd /var/lib/asterisk/agi-bin/
wget https://raw.githubusercontent.com/tanvirnabil/googletts/main/googletts.agi
```

### 3. Set permissions

```bash
chown asterisk:asterisk googletts.agi
chmod +x googletts.agi
```

---

## Test It in Asterisk Dialplan

Edit `/etc/asterisk/extensions_custom.conf` and add:

```
[from-internal-custom]
exten => 1235,1,Answer()
exten => 1235,n,agi(googletts.agi,"This is a simple google text to speech test in english.",en)
exten => 1235,n,hangup
```

Then reload the dialplan:

```bash
asterisk -rx "dialplan reload"
```

Dial extension **1235** from any SIP phone to test.
