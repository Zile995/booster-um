# booster-um
Booster UKI Manager - Simple bash script to manage UKI files generated by booster and systemd-ukify.
The script can only be used on Arch Linux.

## Dependencies:
 * Required:
   * booster - Generate initramfs
   * go-yq - Parse booster and booster-um configs
   * system-ukify - Generate UKI files
   * util-linux - Miscellaneous system utilities for Linux
 * Optional:
   * sbctl - Sign UKI files
   * efibootmgr - Create EFI entries

## Default configuration
* booster-um by default:
  * Detects `XBOOTLDR` partition
  * Signs generated UKI files with sbctl **if installed**
  * Uses default splash `/usr/share/systemd/bootctl/splash-arch.bmp`
  * Copies the current cmdline if `/etc/kernel/cmdline` does not exist
  * Uses default configuration if `/etc/booster-um.yaml` config **is not valid**
  * Generates UKI in `esp/EFI/Linux` dir. If this directory does not exist, it will be created
  * Removes `vmlinuz-*` and `booster-*` leftovers in `/boot` dir (**microcode** image will not be deleted)
  * Removes known EFI entries (Example: `\EFI\LINUX\ARCH-LINUX.EFI`) **if efibootmgr is installed**
  * Will not create separate fallback images if `universal` flag is enabled in `/etc/booster.yaml` config
  * Will not generate EFI entries or fallback UKI files, by default. EFI entries and fallback UKI files are treated as leftovers.
  * Will not regenerate initramfs files on kernel and extramodule installation or upgrades (they are already generated by **booster hook**)

* booster-um libalpm hooks by default:
  * Remove EFI entry on kernel removal
  * Remove UKI file from ESP and sbctl database on kernel removal
  * Regenerate UKIs for all installed kernels when booster, microcode, dkms or firmware files are installed, updated, or removed
  * Will not sign generated UKI files on microcode, extramodules (nvidia, nvidia-lts etc.) and kernel updates, sbctl hook will do that
  * Will sign newly created UKI files if they do not exists in the sbctl database. Also you don't need to manually add UKI files in the sbctl database.

* Default `booster -G` output and `/boot`|`/efi` content:
  * <details>
    <summary>Generating UKI for all kernels</summary>

    ![generation](https://github.com/Zile995/booster-um/assets/32335484/77ef01ee-a044-4f53-964a-b4748579d43c)

    </details>
  * <details>
    <summary>/boot and /efi content</summary>
    
    ![path](https://github.com/Zile995/booster-um/assets/32335484/ef21face-a364-4cee-b4d2-60e5a60187be)

    </details>

## Config file
booster-um config file is located at `/etc/booster-um.yaml`. By default it is empty, here is a default configuration:
 ```YAML
 sign_uki: true
 remove_leftovers: true
 generate_fallback: false

 # EFISTUB 
 efistub: false
 efistub_config:
   preserve_boot_order: true
 ```

* `sign_uki` manages the UKI signing
* `remove_leftovers` manages the removal of leftovers. Besides the vmlinuz and booster files, EFI entries and fallback images are treated as leftovers, they will be removed if `efistub` or `generate_fallback` options are disabled. To trigger new leftovers removal configuration, simply run the `booster-um -G`
* `generate_fallback` manages the creation of fallback (universal) UKI files. Separate fallback images will not be created if `universal` flag is enabled in `/etc/booster.yaml` config
* `efistub` manages EFI entries. If enabled, booster-um will create a new EFI entry. Also, the booster-um will by default preserve old boot order and add the newly created EFI entry at the **end** of the boot order.
* `efistub_config` node provides additional efistub configuration. It is currently possible to control addition of newly created efi entries to the current boot order
* `preserve_boot_order` as mentioned, preserves old boot order. If enabled, newly created EFI entries will be added to the end of the boot order, otherwise they will be added to the beginning
