# How to Install SSL Certificate for Brave Browser

This is probably Brave’s Linux certificate importer, not your password. Chromium’s official fallback is importing directly into its NSS database.

First verify the `.p12` itself:

```bash
openssl pkcs12 -info -noout -in /path/to/device.p12
```

If it accepts the password and prints certificate information, the package is valid.

Then completely close Brave and run:

```bash
sudo apt install libnss3-tools
```

Select Brave’s NSS database:

```bash
if [ -f "$HOME/.pki/nssdb/cert9.db" ]; then
    NSS_DB="$HOME/.pki/nssdb"
else
    NSS_DB="$HOME/.local/share/pki/nssdb"
fi

mkdir -p "$NSS_DB"
```

If `cert9.db` does not exist, initialize it:

```bash
test -f "$NSS_DB/cert9.db" || certutil -N --empty-password -d "sql:$NSS_DB"
```

Import the Autolight CA:

```bash
certutil -d "sql:$NSS_DB" \
  -A -t "C,," \
  -n "Autonomous Light CA" \
  -i /path/to/autolight-ca.crt
```

Import the client identity:

```bash
pk12util -d "sql:$NSS_DB" -i /path/to/device.p12
```

It will ask for the `.p12` export password. Verify the import:

```bash
certutil -d "sql:$NSS_DB" -L
```

Reopen Brave and visit:

```text
https://autolight.local
```

Chromium now defaults to `$HOME/.local/share/pki/nssdb` from Chromium M146 onward, but continues using `$HOME/.pki/nssdb` when an older database already exists. These commands follow Chromium’s current official Linux certificate instructions. [Chromium Linux certificate management](https://chromium.googlesource.com/chromium/src.git/%2B/dd6626cdd58e0ccd306599507de9a47e5a2573e9/docs/linux/cert_management.md)

If `pk12util` fails, send me its exact error text. It distinguishes an unsupported package, duplicate certificate, damaged file, and incorrect NSS database. If the commands succeed but Brave does not see the certificate, also tell me whether Brave was installed through APT, Flatpak, or Snap.