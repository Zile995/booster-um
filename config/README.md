# Config

booster-um config file is located at `/etc/booster-um.yaml`. It is empty by default, here is an example configuration:

 ```YAML
 sign_uki: true
 remove_leftovers: true
 generate_fallback: false
 
 efistub: false
 efistub_config:
   default_entry: linux
   append_entries: true

 sbsign: false
 sbsign_config:
   pcr_banks: sha1,sha256,sha384,sha512
   pcr_private_key: /path/to/pcr-private-key.pem
   pcr_public_key: /path/to/pcr-public-key.pem
   secureboot_private_key: /path/to/DB.key
   secureboot_certificate: /path/to/DB.crt
 ```

* `sign_uki` manages the UKI signing. If enabled, `sbctl` (or `sbsign`), will sign generated UKI files. If it is not specified, its value is set to `true`

* `remove_leftovers` manages the removal of leftovers when generating the UKI files. Besides the vmlinuz and booster files, EFI entries, fallback images and kernel cmdlines are treated as leftovers, they will be removed if `efistub`, `cmdline_per_kernel`, `generate_fallback` options are disabled. If enabled, leftovers will always be removed after generating UKI files. Leftovers will always be removed if you manually delete the UKI for the specified kernel or all installed kernels (`booster-um -r <package>` or `booster-um -R`/`booster-um -C`). If it is not specified, its value is set to `true`

* `generate_fallback` manages the creation of fallback (universal) UKI files. Separate fallback images will not be created if `universal` flag is enabled in the `/etc/booster.yaml` config. If it is not specified, its value is set to `false`

* `cmdline` is the default kernel cmdline. By default all kernels use this cmdline. This option must be defined. Kernel parameters can be written in multiple lines.

## Kernel config

* `kernel_config` node provides kernel configuration:
  * `cmdline_per_kernel` manages the creation of the cmdline per kernel. If enabled, `booster-um` will use the kernel parameters from boosterm-um config file, under the pkgbase `cmdline` option. This option must be defined under the specified pkgbase name. `pkgbase` is the name of the pacman kernel package name (linux, linux-lts, linux-zen etc.) If the kernel cmdline is not specified inside the config file, booster-um will look for the `/etc/kernel/$pkgbase.cmdline` or `/etc/kernel/$pkgbase-fallback.cmdline` file. If it is not specified, its value is set to `false`

  * `share_default_cmdline` allows the default cmdline to be shared with the cmdline of the specified kernel. *The default cmdline `cmdline`, `fallback_cmdline` or the `/etc/kernel/cmdline`, `/etc/kernel/fallback-cmdline` file will be used as a shared cmdline for all kernels. If it is not specified, its value is set to `false`
 
  * `default_splash` a picture to display during boot. This is the default splash for all unspecified pkgbase kernels under `kernel_config` node. The argument is a path to a BMP file. The default `/usr/share/systemd/bootctl/splash-arch.bmp` picture will be used if this path is invalid or not specified. You can disable splash screen

  * `default_initramfs` provides initramfs type configuration for all other unspecified kernels. Under this node, you can specify up to two types, `default` and `fallback`. By default this array has `default` and `fallback` types.
  * > **Note**: If you specified `fallback` type, you must enable `generate_fallback`, otherwise it will generate `default` images.

## EFISTUB config

* `efistub` manages EFI entries. If enabled, `booster-um` will create a new EFI entry. If it is not specified, its value is set to `false`
* `efistub_config` node provides additional efistub configuration:
  * `default_entry` makes sure that the EFI entry of the specified kernel is the first in the EFI boot order. If fallback UKI is generated for the specified kernel, its EFI entry will be added after the default entry. After changing its value, it is enough to regenerate all images (`booster-um -G`)  
  * `append_entries` takes care of where new EFI entries will be added to the boot order. If enabled , **newly** created EFI entries will be added to the end of the boot order, otherwise they will be added to the beginning. If it is not specified, its value is set to `true`

## sbsign config

* `sbsign` manages UKI signing using the `sbsign` tool. If enabled, `sbsign` will be used instead of `sbctl`. After enabling this type of signing, the options in the `sbsign_config` node can be set arbitrarily. If it is not specified, its value is set to `false`
* `sbsign_config` node provides `sbsign` configuration:
  * `pcr_banks` a comma separated list of PCR banks to sign a policy for
  * `pcr_private_key` a path to a private key to use for signing PCR policies
  * `pcr_public_key` a path to public key to use for signing PCR policies
  * `secureboot_private_key` a path to a private key to use for signing of the UKI file
  * `secureboot_certificate` a path to a certificate to use for signing of UKI file
