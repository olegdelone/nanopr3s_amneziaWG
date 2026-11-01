# Disclaimer 
Everything was done via chatgpt. We compiled sources on Ubuntu wsl, and it works on my nanoPi r3s + FriendlyWrt 25.12.5 installed on it, which I bought 3 month ago.
It is not ready-to-go apk, but having spent several hours to get it done, i decided to grab everything together into this instrunction - as a long story short of the whole AI context, so that it may have mistakes, or missing files\steps.

# AmneziaWG for FriendlyElec NanoPi R3S

A backup installation bundle for AmneziaWG on **FriendlyWrt 25.12.5**, running on the **FriendlyElec NanoPi R3S LTS**.

> **Important:** This bundle was prepared for kernel version `6.1.141` and the `aarch64` architecture. The included kernel module must not be assumed compatible with any other kernel version.

## Platform specifications

| Parameter | Value |
|---|---|
| Device | FriendlyElec NanoPi R3S LTS |
| Firmware | FriendlyWrt 25.12.5 |
| Architecture | aarch64 |
| Kernel version | 6.1.141 |
| AWG version | 3.1.20260812 |

## Bundle contents

```text
AmneziaWG-NanoPi-R3S/
├── kernel-module/
│   └── amneziawg.ko
├── awg-tools/
│   └── awg
├── luci-integration/
│   ├── lib/netifd/proto/amneziawg.sh
│   ├── usr/bin/amneziawg_watchdog
│   ├── usr/share/rpcd/
│   └── www/luci-static/resources/
└── build-info/
    └── build-info.txt
```

- `amneziawg.ko` — AmneziaWG kernel module.
- `awg` — ARM64 AmneziaWG command-line utility.
- `luci-integration` — integration files for netifd, rpcd and LuCI.
- `build-info` — build and target-system information.

## Installation

### 1. Check the kernel version

Connect to the router over SSH:

```sh
ssh root@192.168.2.1
```

Check the running kernel version:

```sh
uname -r
```

It must be `6.1.141`. If the version is different, **do not install the included kernel module**.

### 2. Copy the files to the router

Extract the archive on your computer. From PowerShell, run:

```powershell
scp -r .\AmneziaWG-NanoPi-R3S root@192.168.2.1:/tmp/
```

Replace `192.168.2.1` with your router's IP address if necessary.

### 3. Install the kernel module

Connect to the router and enter the bundle directory:

```sh
ssh root@192.168.2.1
cd /tmp/AmneziaWG-NanoPi-R3S
```

Copy the module into the kernel modules directory:

```sh
mkdir -p /lib/modules/$(uname -r)/extra
cp kernel-module/amneziawg.ko /lib/modules/$(uname -r)/extra/
depmod -a
```

If the module is not already loaded, load it:

```sh
insmod /lib/modules/$(uname -r)/extra/amneziawg.ko
```

Enable module loading at boot:

```sh
mkdir -p /etc/modules.d
echo amneziawg > /etc/modules.d/amneziawg
```

### 4. Install the AWG utility

```sh
cp awg-tools/awg /usr/bin/awg
chmod 755 /usr/bin/awg
```

### 5. Install LuCI/netifd integration

From the bundle's root directory, run:

```sh
cp -a luci-integration/. /
```

Set executable permissions:

```sh
chmod 755 /usr/bin/amneziawg_watchdog
chmod 755 /lib/netifd/proto/amneziawg.sh
```

### 6. Restart services

```sh
/etc/init.d/rpcd restart
/etc/init.d/network restart
```

Restarting the network may interrupt your SSH connection. Reconnect to the router if necessary.

## Verification

Check whether the kernel module is loaded:

```sh
lsmod | grep amneziawg
```

Check the AWG utility:

```sh
awg --version
```

Check whether the protocol handler is registered:

```sh
ubus call network get_proto_handlers
```

The output should include the `amneziawg` protocol.

After installation, the AmneziaWG protocol should also be available when adding a network interface in LuCI.

## Important notes

- Do not use `opkg --force` to install incompatible packages.
- Do not install the kernel module on a different kernel version.
- After upgrading FriendlyWrt, verify kernel-module compatibility. A kernel update may require rebuilding the module.
- This bundle is intended for manual installation. It does not include an automatically configured VPN tunnel, peer settings, private keys or firewall configuration.
- Never publish private keys, personal VPN configuration files or other secrets in a public GitHub repository.

## Licenses

When redistributing this bundle, comply with the licenses of the AmneziaWG, OpenWrt/FriendlyWrt and integration projects. Include the required license notices and links to the original source repositories.
