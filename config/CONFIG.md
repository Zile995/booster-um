# Config

booster-um config file is located at `/etc/booster-um.yaml`. It is empty by default, but here is an example configuration:

 ```YAML
 sign_uki: true
 sbsign: false
 efistub: true
 enable_splash: true
 remove_leftovers: true
 generate_fallback: true

 cmdline: >
   root=LABEL=arch_root
   rw
   quiet
   loglevel=3
   udev.log_level=3
   sysrq_always_enabled=1
   vt.global_cursor_default=0

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
   linux-lts:
     splash: false
     # cmdline:
     # fallback_cmdline:
     initramfs: [fallback]
   # linux-zen:
 
 efistub_config:
   default_entry: linux
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
efistub: true
enable_splash: true
remove_leftovers: true
generate_fallback: true

cmdline: >
  root=LABEL=arch_root
  rw
  quiet

fallback_cmdline: "root=LABEL=arch_root rw"
```

* `sign_uki` manages the UKI signing. If enabled, `sbctl` (or `sbsign`), will sign generated UKI files. If it is not specified, its value is set to `true`

* `sbsign` manages UKI signing using the `sbsign` tool. If enabled, `sbsign` will be used instead of `sbctl`. After enabling this type of signing, the options in the `sbsign_config` node can be set arbitrarily. If it is not specified, its value is set to `false`

* `efistub` manages EFI entries. If enabled, `booster-um` will create a new EFI entry. If it is not specified, its value is set to `false`

* `remove_leftovers` manages the removal of leftovers when generating the UKI files. Besides the vmlinuz and booster files, EFI entries, fallback images and kernel cmdlines are treated as leftovers, they will be removed if `efistub`, `cmdline_per_kernel`, `generate_fallback` options are disabled. If enabled, leftovers will always be removed after generating UKI files. Leftovers will always be removed if you manually delete the UKI for the specified kernel or all installed kernels (`booster-um -r <package>` or `booster-um -R`/`booster-um -C`). If it is not specified, its value is set to `true`

* `generate_fallback` manages the creation of fallback (universal) UKI files. Separate fallback images will not be created if `universal` flag is enabled in the `/etc/booster.yaml` config. If it is not specified, its value is set to `false`

* `enable_splash` is an option that enables splash screen. If you want to disable splash for **all specified or unspecified** kernels under `kernel_config` node, set it to `false`. By default this option is enabled and `/usr/share/systemd/bootctl/splash-arch.bmp` splash will be used (you can change it with `default_splash` option under `kernel_config` node)

* `cmdline` is the default kernel cmdline and is used by **all** kernels. If `cmdline` is not defined here, booster-um will try to use the cmdline from `/etc/kernel/cmdline` file. If cmdline is not defined neither in the config nor in the `/etc/kernel/cmdline` file, the current cmdline from `/proc/cmdline` will be used. Kernel parameters can be written in multiple lines after the `>` sign. For example:
  ```YAML
  cmdline: >
    root=LABEL=arch_root
    rw
    quiet
  ```
  
* `fallback_cmdline` is same as `cmdline` but for fallback kernel images. If it is not defined here, booster-um will try to use the cmdline from `/etc/kernel/fallback-cmdline` file. If cmdline is not defined neither in the config nor in the `/etc/kernel/fallback-cmdline` file, the current cmdline from `/proc/cmdline` will be used

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
   linux-lts:
     splash: false
     cmdline: "root=LABEL=arch_root rw"
     fallback_cmdline: "root=LABEL=arch_root rw"
     initramfs: [fallback]
```

> `pkgbase` = kernel package name (linux, linux-lts, linux-zen, etc.)

`kernel_config` node provides kernel configuration:

  * `cmdline_per_kernel` manages the creation of the cmdline per kernel. If this option is enabled, each kernel will use a separate cmdline which can be defined under `pkgbase` node within the `cmdline` option. If it is not specified, its value is set to `false`

  * `share_default_cmdline` allows default cmdline to be shared with the cmdline of the **specified** kernel pkgbase under the `kernel_config` node. The default cmdline `cmdline`, `fallback_cmdline` or `/etc/kernel/cmdline`, `/etc/kernel/fallback-cmdline` files will be used as a shared cmdline for **all** kernels. That means that the kernel cmdline specified under `pkgbase` node, will be added to the default cmdline. This option only takes effect if the `cmdline_per_kernel` option is enabled. By default this option is set to `false`

  * `default_initramfs` array provides initramfs type configuration for all other unspecified kernels. You can specify up to two types, `default` and `fallback`. if not defined, its values ​​will be `default` and `fallback`
   > **Note**: If you specified `fallback` type, you must enable `generate_fallback`, otherwise it will generate `default` images only
 
  * `default_splash` a picture to display during boot. This is the default splash for all **unspecified** kernels under `kernel_config` node. The argument is a path to a BMP file. The default `/usr/share/systemd/bootctl/splash-arch.bmp` picture will be used if this path is invalid or not specified. To disable splash screen for all **unspecified pkgbases** under `kernel_config` node, simply set this option to `false` or leave it blank, for example:
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

    * `cmdline` is the cmdline for the **specified** kernel. This cmdline will be applied if the `cmdline_per_kernel` option is enabled. If `cmdline` is not defined here, booster-um will try to use the cmdline from `/etc/kernel/$pkgbase.cmdline` file. If cmdline is not defined neither in the config nor in the `/etc/kernel/$pkgbase.cmdline`, booster-um will try to use the default `cmdline` outside the `kernel_config` node

    * `fallback_cmdline` is same as `cmdline` but for fallback kernel images. If it is not defined here, booster-um will try to use the cmdline from `/etc/kernel/$pkgbase-fallback.cmdline` file. If cmdline is not defined neither in the config nor in the `/etc/kernel/$pkgbase-fallback.cmdline`, booster-um will try to use the default `fallback_cmdline` outside the `kernel_config` node

    * `initramfs` provides initramfs type configuration for **specified** kernel. Here you can specify up to two types, `default` and `fallback`. If it is not defined, the `default_initramfs` will be used

## EFISTUB config

```YAML
efistub_config:
 default_entry: linux
 append_entries: true
```

* `efistub_config` node provides additional efistub configuration:
  * `default_entry` makes sure that the EFI entry of the **specified** kernel is the first in the EFI boot order. If fallback UKI is generated for the specified kernel, its EFI entry will be added after the default entry. After changing its value, it is enough to regenerate all images (`booster-um -G`)  
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
