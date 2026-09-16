# How to switch CUDA Version to a different downloaded version:

Switching CUDA versions on Ubuntu requires multiple CUDA toolkits installed side-by-side in `/usr/local/` (e.g., `/usr/local/cuda-11.8`, `/usr/local/cuda-12.1`). Choose the method that best matches your workflow:

### First check the installed CUDA Versions:

To check which CUDA versions are installed on your Ubuntu system, use these terminal commands depending on what you are looking for:

**1. List all installed CUDA directories (Best for side-by-side installations)**
Shows all CUDA toolkit versions installed in the default directory:

```bash
ls -d /usr/local/cuda*

```

**2. Check the active compiler version (`nvcc`)**
Shows the CUDA version currently active in your system `PATH`:

```bash
nvcc --version

```

**3. Check via `dpkg` (If installed via APT)**
Lists all CUDA packages managed by the Debian package manager:

```bash
dpkg -l | grep cuda

```

**4. Check current symbolic link target**
Shows which specific version the main `/usr/local/cuda` link currently points to:

```bash
readlink -f /usr/local/cuda

```

**5. Check the active NVIDIA driver & supported CUDA version**
Shows the NVIDIA driver status and the *maximum* CUDA version supported by that driver:

```bash
nvidia-smi

```

### Method 1: Update the Symlink (System-Wide Switch)

If your system relies on the `/usr/local/cuda` symbolic link, re-point it to your target version.

1. **Remove the existing link and create a new one:**
```bash
sudo rm /usr/local/cuda
sudo ln -s /usr/local/cuda-12.1 /usr/local/cuda

```


*(Replace `cuda-12.1` with your desired target version)*.
2. **Verify environment paths in `~/.bashrc`:**
Ensure your shell points to `/usr/local/cuda` rather than a specific version number:
```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

```


3. **Reload your profile:**
```bash
source ~/.bashrc

```



---

### Method 2: Update Environment Variables (Per-User or Per-Session)

To switch CUDA versions for a single terminal session without modifying root settings:

1. **Set the environment paths in your terminal:**
```bash
export CUDA_HOME=/usr/local/cuda-11.8
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH

```


2. **To make it permanent for your user account:**
Add those lines directly to `~/.bashrc` and run `source ~/.bashrc`.

---

### Method 3: Use `update-alternatives` (Debian Native)

If CUDA was installed via APT and managed by Debian's alternative system:

1. **Configure alternatives for CUDA:**
```bash
sudo update-alternatives --config cuda

```


2. **Select the number corresponding to your preferred version** from the interactive menu.

---

### Verification

Verify that the active CUDA compiler matches your target version:

```bash
nvcc --version

```