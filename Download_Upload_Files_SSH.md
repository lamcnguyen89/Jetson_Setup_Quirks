# Downloading and Uploading Files Over SSH

Run these from your **local** machine unless noted.

## Upload (local -> remote)

```bash
scp file.txt user@host:/remote/path/
scp -r mydir user@host:/remote/path/
rsync -avhP mydir/ user@host:/remote/path/mydir/
```

## Download (remote -> local)

```bash
scp user@host:/remote/path/file.txt ./
scp -r user@host:/remote/path/mydir ./
rsync -avhP user@host:/remote/path/mydir/ ./mydir/
```

## Options

- Custom port: `scp -P 2222 ...` (capital P) or `rsync -e "ssh -p 2222" ...`
- Key file: `scp -i ~/.ssh/id_ed25519 ...`
- Resume interrupted transfer: re-run the same `rsync -avhP` command.
- Compress over slow links: `scp -C` / `rsync -z`.
- Preview rsync without copying: add `-n` (dry run).
- Paths with spaces: quote them, e.g. `scp "user@host:/path/my file.txt" ./`

## Interactive: SFTP

```bash
sftp user@host
sftp> ls / cd / lcd        # remote ls/cd, local lcd
sftp> put file.txt         # upload
sftp> get file.txt         # download
sftp> put -r mydir / get -r mydir
sftp> exit
```

## Tips

- Trailing `/` on an rsync source copies its contents; without it, the folder itself.
- Add `--delete` to rsync only if you want the destination to mirror the source (it deletes extra files).
- Use `~/.ssh/config` host aliases to shorten commands: `scp file.txt jetson:/tmp/`
- For long transfers, use `tmux`/`screen` on the machine running the command.
