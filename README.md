# brother-mfc-7860dw

Fork of the [brother-mfc-7860dw AUR package](https://aur.archlinux.org/packages/brother-mfc-7860dw), with local packaging and driver-configuration changes.

This fixes an issue in the upstream where individual job settings (duplex, DPI, etc.) were not getting applied.  The root cause was a permissions issue on the `brMFC7860DWrc` file.

This package also installs into `/usr/local/Brother/` to match the upstream RPM.  While this does conform to [Arch Package Guidelines](https://manual.archlinux.page/package-guidelines/), it avoids complexity since the upstream RPM hardcodes some paths to `/usr/local`.

The vendor installer is not run during packaging; the PPD and CUPS filter are installed as package files. The packaged default RC configuration enables duplex printing and selects Letter paper.

## Build and install

From the repository directory on Arch Linux, run:

```sh
makepkg -si
```

Run `makepkg` as a regular user, not with `sudo`. The `-s` option installs any missing dependencies, and `-i` installs the package after it has been built.

## Runtime configuration permissions

`brcupsconfig4` applies job options by renaming and rewriting `inf/brMFC7860DWrc`. The package therefore makes that RC file writable by the `cups` group and makes its containing `inf` directory group-writable. The file remains root-owned and is not world-writable.

The RC file is shared mutable state. Locking is not implemented in this package version. Concurrent print jobs may race while changing or reading settings; serializing the helper and filter use of this file is a future consideration.

## Migration from the old relocated package

If upgrading from a version that used `/usr/share/brother/printer`, check for the manually created compatibility symlink at `/usr/local/Brother/Printer/MFC7860DW`. Remove that symlink before installing this package's real vendor tree at the same path. Verify it is the expected symlink before removing it; do not remove a real directory or unrelated local data.
