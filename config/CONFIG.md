# Config

booster-um config file is located at `/etc/booster-um.yaml`. It is empty by default, but here is an example configuration:

 ```YAML
 sign_uki: true
 sbsign: false
 colors: true
 efistub: false
 microcode: true
 verbose: false
 low_memory: false
 enable_splash: true
 remove_leftovers: true
 generate_fallback: false
 append_machine_id: false
 os_id: "arch"

 cmdline: >
   root=LABEL=arch_root
   rw
   quiet
   loglevel=3
   udev.log_level=3
   sysrq_always_enabled=1

 fallback_cmdline: >
   root=LABEL=arch_root
   rw
   sysrq_always_enabled=1

 kernel_config:
   cmdline_per_kernel: false
   share_default_cmdline: false
   default_initramfs: [default, fallback]
   default_splash: /usr/share/systemd/bootctl/splash-arch.bmp
   linux:
     # splash:
     # cmdline:
     # fallback_cmdline:
     initramfs: [default]
     booster_config: |
       compression: zstd
       extra_files: busybox,fsck,fsck.ext4
   linux-lts:
     splash: false
     # cmdline:
     # fallback_cmdline:
     initramfs: [fallback]
   # linux-zen:
 
 efistub_config:
   default_entry: linux
   os_name: "Arch Linux"
   append_entries: true

 sbsign_config:
   pcr_banks: sha1,sha256,sha384,sha512
   pcr_private_key: /path/to/pcr-private-key.pem
   pcr_public_key: /path/to/pcr-public-key.pem
   secureboot_private_key: /path/to/DB.key
   secureboot_certificate: /path/to/DB.crt
 ```

## First level configuration

```YAML
sign_uki: true
sbsign: false
colors: true
efistub: false
microcode: true
verbose: false
low_memory: false
enable_splash: true
remove_leftovers: true
generate_fallback: false
append_machine_id: false
os_id: "arch"

cmdline: >
  root=LABEL=arch_root
  rw
  quiet

fallback_cmdline: "root=LABEL=arch_root rw"
```

* `sign_uki` manages the UKI signing. If enabled, `sbctl` (or `sbsign`), will sign generated UKI files. If it is not specified, its value is set to `true`

* `sbsign` manages UKI signing using the `sbsign` tool. If enabled, `sbsign` will be used instead of `sbctl`. After enabling this type of signing, the options in the `sbsign_config` node can be set arbitrarily. If it is not specified, its value is set to `false`

* `colors` colors the output of the booster-um. If it is not specified, its value is set to `true`

* `efistub` manages EFI entries. If enabled, booster-um will create a new EFI entry. If it is not specified, its value is set to `false`

* `microcode` manages the inclusion of the microcode images (intel-ucode and amd-ucode). If it is not specified, its value is set to `true`

* `verbose` enables verbose output for `booster` during initramfs generation. When enabled, `low_memory` mode is automatically activated to build initramfs sequentially and prevent log output from overlapping. If it is not specified, its value is set to `false`

* `low_memory` is an option that prevents high memory usage, especially when fallback images are generated. Instead of generating initramfs in parallel, booster-um will generate initramfs one by one. This is certainly slower, but takes up less memory. If it is not specified, its value is set to `false`

* `enable_splash` is an option that enables splash screen. If you want to disable splash for **all specified or unspecified** kernels under `kernel_config` node, set it to `false`. By default this option is enabled and `/usr/share/systemd/bootctl/splash-arch.bmp` splash will be used (you can change it with `default_splash` option under `kernel_config` node)

* `remove_leftovers` manages the removal of leftovers when generating the UKI files. Besides the vmlinuz and booster files: EFI entries, fallback images and kernel cmdlines are also treated as leftovers. They will be checked if `efistub`, `cmdline_per_kernel` or `generate_fallback` options are changed. If enabled, leftovers will always be removed after generating UKI files. Leftovers will always be removed if you manually delete the UKI for the specified kernel, or all installed kernels (`booster-um -r <package>` or `booster-um -R`/`booster-um -C`). If it is not specified, its value is set to `true`

* `generate_fallback` manages the creation of `fallback` (universal) UKI files. The fallback images will only be generated if `universal` flag is enabled in the `/etc/booster.yaml` config, so the main `/etc/booster.yaml` config file is respected. If it is not specified, its value is set to `false`

* `append_machine_id` appends the system machine ID (`/etc/machine-id`) to the generated UKI filename (e.g., `/EFI/Linux/<os_id>-<kernel_package>-<machine_id>[-fallback]`). This can be particularly useful when dual-booting two installations of the same distribution on a shared ESP to prevent file collisions. If it is not specified, its value is set to `false`

* `os_id` sets a custom OS identifier prefix for UKI filenames (`/EFI/Linux/<os_id>-<kernel_package>.efi`). If not set, the value of `ID` from `/etc/os-release` (e.g. `arch`) is used as the default. This option defines the prefix appended before the kernel package name in generated Unified Kernel Image (UKI) filenames: (e.g. /EFI/Linux/**arch**-linux.efi )

* `cmdline` is the default kernel cmdline and is used by **all** kernels. If `cmdline` is not defined here, booster-um will try to use the cmdline from `/etc/kernel/cmdline` file. If cmdline is not defined neither in the config nor in the `/etc/kernel/cmdline` file, the current cmdline from `/proc/cmdline` will be used. Kernel parameters can be written in multiple lines after the `>` sign. For example:
  ```YAML
  cmdline: >
    root=LABEL=arch_root
    rw
    quiet
  ```
Also, you can write kernel parameters in multiple lines inside the cmdline files located in `/etc/kernel`. All sequences of whitespaces (newlines, tabs, spaces) will be replaced with single space.

* `fallback_cmdline` is same as `cmdline` but for fallback kernel images. If the cmdline is not defined here, booster-um will try to use the cmdline from files in this order:
  * `/etc/kernel/cmdline-fallback` 
  * `/etc/kernel/cmdline` 
  
    If cmdline is not defined neither in the config nor in the mentioned files, the current cmdline from `/proc/cmdline` will be used

## Kernel config (`kernel_config` node)

```YAML
 kernel_config:
   cmdline_per_kernel: false
   share_default_cmdline: false
   default_initramfs: [default, fallback]
   default_splash: /path/to/splash.bmp
   linux: 
     cmdline: >
       root=LABEL=arch_root
       rw
       quiet
     fallback_cmdline: "root=LABEL=arch_root rw"
     initramfs: [default]
     booster_config: |
       compression: zstd
       extra_files: busybox,fsck,fsck.ext4
   linux-lts:
     splash: false
     cmdline: "root=LABEL=arch_root rw quiet"
     fallback_cmdline: "root=LABEL=arch_root rw"
     initramfs: [fallback]
```

> `pkgbase` = kernel package name (linux, linux-lts, linux-zen, etc.)

`kernel_config` node provides kernel configuration:

  * `cmdline_per_kernel` manages the creation of the cmdline per kernel. If this option is enabled, each kernel will use a separate cmdline which can be defined under `pkgbase` node within the `cmdline` option. If it is not specified, its value is set to `false`

  * `share_default_cmdline` allows default cmdline to be shared with the cmdline of the **specified** kernel pkgbase under the `kernel_config` node. The default cmdline `cmdline`, `fallback_cmdline` or `/etc/kernel/cmdline`, `/etc/kernel/cmdline-fallback` files will be used as a shared cmdline for **all** kernels. That means that the kernel cmdline specified under `pkgbase` node, will be added to the default cmdline. This option only takes effect if the `cmdline_per_kernel` option is enabled. By default this option is set to `false`

  * `default_initramfs` array provides initramfs type configuration for all other unspecified kernels. You can specify up to two types, `default` and `fallback`. If not defined, its values ​​will be `default` and `fallback`
   > **Note**: If you specified `fallback` type, you must enable `generate_fallback`, otherwise it will generate `default` images only. If you enable universal inside the booster config file `/etc/booster.yaml`, booster-um will only create fallback images.
 
  * `default_splash` a picture to display during boot. This is the default splash for all **unspecified** kernels under `kernel_config` node. The argument is a path to a **BMP** file. The default `/usr/share/systemd/bootctl/splash-arch.bmp` picture will be used if this path is invalid or not specified. To disable splash screen for all **unspecified pkgbases** under `kernel_config` node, simply set this option to `false` or leave it blank, for example:
    ```YAML
      # valid options
      kernel_config:
        default_splash:
        default_splash: ''
        default_splash: ""
        default_splash: false
    ```

  * `pkgbase` (kernel package name, below as `$pkgbase`) node provides additional configuration for **specified** kernel package name:

    * `splash` a picture to display during boot for the **specified** kernel. To disable splash screen for **specified** kernel pkgbase, simply set this option to `false` or leave it blank. If `splash` is not defined, the `default_splash` option outside the `$pkgbase` node, will be used

    * `cmdline` is the cmdline for the **specified** kernel. This cmdline will be applied if the `cmdline_per_kernel` option is enabled. If cmdline is not defined here, booster-um will try to use the default cmdline in the config file, outside the `kernel_config` node, the `cmdline`. If `cmdline` is not defined here, booster-um will try to use the cmdline from files in this order:
      * `/etc/kernel/$pkgbase-cmdline` 
      * `/etc/kernel/cmdline` 
      If cmdline doesn't exist neither in the config nor in the mentioned files, booster-um will try to use the current cmdline from `/proc/cmdline`

    * `fallback_cmdline` is same as `cmdline` but for fallback kernel images. If cmdline is not defined here, booster-um will try to use the default cmdline for `$pkgbase`. If `$pkgbase` cmdlines are not defined, booster-um will try to use the default cmdlines from the config files, outside the `kernel_config` node (first `fallback_cmdline`, then `cmdline`). If these cmdlines are not defined in the booster-um config file, booster-um will try to use the cmdline from files in this order:
      * `/etc/kernel/$pkgbase-cmdline-fallback` 
      * `/etc/kernel/$pkgbase-cmdline` 
      * `/etc/kernel/cmdline-fallback` 
      * `/etc/kernel/cmdline` 
      
        If cmdline doesn't exist neither in the config file nor in the mentioned files, booster-um will try to use the current cmdline from `/proc/cmdline`

    * `initramfs` provides initramfs type configuration for **specified** kernel. Here you can specify up to two types, `default` and `fallback`. If it is not defined, the `default_initramfs` will be used

    * `booster_config` provides additional booster configuration for **specified** kernel. Booster configuration can be written in multiple lines after the `|` sign. You can read [here](https://github.com/anatol/booster/blob/master/docs/manpage.md#config-file) about booster configuration. For example:
      ```YAML
      booster_config: |
       compression: lz4
       extra_files: busybox,fsck,fsck.ext4
      ```
     > **Note**: If you enable `universal` flag here, booster-um will only create a fallback UKI for the **specified** kernel even if `generate_fallback` is disabled, or `initramfs` (`default_initramfs`) has type specified. So, the booster config is always respected.

## EFISTUB config

```YAML
efistub_config:
 default_entry: linux
 os_name: "Arch Linux"
 append_entries: true
```

* `efistub_config` node provides additional efistub configuration:
  * `default_entry` makes sure that the EFI entry of the **specified** kernel is the first in the EFI boot order. If fallback UKI is generated for the specified kernel, its EFI entry will be added after the default entry. After changing its value, it is enough to regenerate all images (`booster-um -G`)  
  * `os_name` sets a custom OS display name prefix for EFI boot entry labels (`<os_name> (<kernel_package>)`). If not set, the value of `NAME` from `/etc/os-release` (e.g., `Arch Linux`) is used as the default. This option defines the OS display name prefix that appears before the kernel package name in EFI boot entry labels (e.g., **Arch Linux** (linux), **My Arch** (linux))
  * `append_entries` takes care of where new EFI entries will be added to the boot order. If enabled , **newly** created EFI entries will be added to the end of the boot order, otherwise they will be added to the beginning. If it is not specified, its value is set to `true`

## sbsign config

```YAML
 sbsign_config:
   pcr_banks: sha1,sha256,sha384,sha512
   pcr_private_key: /path/to/pcr-private-key.pem
   pcr_public_key: /path/to/pcr-public-key.pem
   secureboot_private_key: /path/to/DB.key
   secureboot_certificate: /path/to/DB.crt
```

* `sbsign_config` node provides `sbsign` configuration:
  * `pcr_banks` a comma separated list of PCR banks to sign a policy for
  * `pcr_private_key` a path to a private key to use for signing PCR policies
  * `pcr_public_key` a path to public key to use for signing PCR policies
  * `secureboot_private_key` a path to a private key to use for signing of the UKI file
  * `secureboot_certificate` a path to a certificate to use for signing of UKI file
