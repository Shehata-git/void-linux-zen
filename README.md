# Void Linux Zen Kernel (with Binder Support)

A custom Void Linux kernel package based on the `linux-zen` patchset, specifically configured with Binder modules enabled (ideal for Waydroid compatibility).

---

# Using the Pre-compiled Repository

If you want to skip the lengthy compilation process and just install the working kernel, you can use the pre-built packages hosted in this repository's releases.

*Note: If your current kernel works perfectly fine and you don't specifically need Zen or Waydroid/Binder support, there's no strict need to switch. This is provided for convenience and personal use.*

### 1. Trust the Repository Key
XBPS requires the public RSA key to verify the integrity of the downloaded packages. Download it directly into your system's XBPS keys directory:
```bash
sudo mkdir -p /var/db/xbps/keys
sudo curl -Lo /var/db/xbps/keys/star-repo.pub https://raw.githubusercontent.com/Shehata-git/void-linux-zen/main/star-repo.pub

```

### 2. Add the Repository Configuration

Create a configuration file in `/etc/xbps.d/` pointing to the GitHub release tag that contains the compiled binaries.

```bash
echo "repository=https://github.com/Shehata-git/void-linux-zen/releases/download/v6.19.8_1" | sudo tee /etc/xbps.d/50-star-linux-zen.conf

```

### 3. Sync and Install

Update your repository indexes and install the custom kernel along with its headers:

```bash
sudo xbps-install -S linux-zen6.19 linux-zen6.19-headers

```

Reboot your system to start using the new kernel.

---

# Compiling from source

## Prerequisites
Ensure your host system has the base development tools, git, and `xtools` (which provides the `xgensum` and `xi` utilities) installed before starting.
```bash
sudo xbps-install -S base-devel xtools git
```

### 1. Prepare the Build Environment

Clone the official `void-packages` repository and bootstrap the isolated build environment.

```bash
git clone --depth 1 [https://github.com/void-linux/void-packages.git](https://github.com/void-linux/void-packages.git)
cd void-packages

./xbps-src binary-bootstrap
```

### 2. Scaffold the Custom Package

Duplicate the existing standard kernel package structure to serve as the base, then set up the required symlinks for the header and debug subpackages.

```bash
cp -r srcpkgs/linux6.19 srcpkgs/linux-zen6.19

ln -s linux-zen6.19 srcpkgs/linux-zen6.19-headers
ln -s linux-zen6.19 srcpkgs/linux-zen6.19-dbg
```

### 3. Inject Binder Configuration

Append the required Android Binder flags to the custom kernel configuration file.

**Target File:** `void-packages/srcpkgs/linux-zen6.19/files/x86_64-dotconfig-custom`

```bash
echo "CONFIG_ANDROID_BINDER_IPC=y" >> srcpkgs/linux-zen6.19/files/x86_64-dotconfig-custom
echo "CONFIG_ANDROID_BINDERFS=y" >> srcpkgs/linux-zen6.19/files/x86_64-dotconfig-custom
```

### 4. Replace the Build Template

Overwrite the default `template` file in the package directory with the custom Zen template provided in this repository.

**Target Path:** `void-packages/srcpkgs/linux-zen6.19/template`
***Manually copy/overwrite the `template` file into this directory before proceeding.***

### 5. Generate Checksums and Compile

Update the package checksums for the new source files and start the compilation.

*Note: During the build process, the kernel configuration script will prompt you for input. Press **Enter** to accept the default Zen configurations for all prompts.*

```bash
xgensum -i srcpkgs/linux-zen6.19/template
./xbps-src pkg linux-zen6.19
```

### 6. Install the Compiled Kernel

Once compilation finishes, install the newly built kernel and headers from your local repository using `xi`.

```bash
sudo xi linux-zen6.19 linux-zen6.19-headers
```

Kernel hooks should automatically update your bootloader. If they do not, or to be absolutely safe, regenerate your GRUB configuration manually:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
