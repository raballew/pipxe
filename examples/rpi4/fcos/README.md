# Fedora CoreOS

## Download

First, navigate to this directory and download the latest stable `aarch64`
version of Fedora CoreOS.

## Official release

```bash
podman run --pull=always --rm -v ${PWD}:/data -w /data quay.io/coreos/coreos-installer:release download -f pxe --architecture aarch64
# use default names
mv fedora-coreos-*-live-initramfs.aarch64.img fedora-coreos-live-initramfs.aarch64.img
mv fedora-coreos-*-live-rootfs.aarch64.img fedora-coreos-live-rootfs.aarch64.img
mv fedora-coreos-*-live-kernel-aarch64 fedora-coreos-live-kernel-aarch64.gz
# piPXE uses EFI which only supports uncompressed kernel artifacts
gzip -d fedora-coreos-live-kernel-aarch64.gz
```

## Custom build

Download the disk image using [oras]:

```bash
podman run -it --rm -v $(pwd):/workspace:Z ghcr.io/oras-project/oras:v0.12.0 pull ghcr.io/raballew/pipxe/fcos:${GIT_REVISION} -a
```

Where:

* `${GIT_REVISION}` -  Full `SHA-1 object name` from main branch, [SemVer]
  compliant `tag` or `latest`

## Customize

Then check that [fake-cpuinfo](fake-cpuinfo) content is equal to your devices
specifications for `Revision` that is usually stored in `/proc/cpuinfo` when
using Raspbian.

Now render the ignition file:

```bash
podman run --pull=always --rm -v ${PWD}:/data:z -w /data quay.io/coreos/butane:release --pretty --strict -d /data/ fcos.bu > fcos.ign
```

Now, place [fcos.ipxe](fcos.ipxe) in the root directory of your `tftp` server.
Then move all downloaded files and `fcos.ign` to your HTTP servers root
directory. Finally, prepare your `http`, `dhcp` and `tftp` servers accordingly,
so that the Raspberry Pi once booted loads [fcos.ipxe](fcos.ipxe) and has access
to all artifacts.

Then follow the steps described [here](../../../README.md#use) and finally power
on your Raspberry Pi.

[oras]: https://github.com/oras-project/oras
