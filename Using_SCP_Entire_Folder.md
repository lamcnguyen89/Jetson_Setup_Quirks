# Example of how to Copy all files and folders to a destination using SSH

This example is how to copy all items in a folder to my batocera installation:

```bash
shopt -s dotglob nullglob
scp -r -- * root@BATOCERA_IP:/userdata/bios/
```

Or to just copy a folder do something like:

```bash
scp -r bios root@batocera_ip:/userdata/bios/
```
