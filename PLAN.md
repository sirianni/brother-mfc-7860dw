# Plan: fix Brother MFC-7860DW CUPS permission issue

## Goal

Allow the unprivileged CUPS filter user to apply per-job printer settings through Brother's `brcupsconfig4` helper, without making the RC file world-writable or running filters as root.

## Packaging direction

Keep the vendor's original `/usr/local/Brother/Printer/MFC7860DW` layout. This is a deliberate choice for this legacy RPM-derived driver: it avoids maintaining path rewrites and a compatibility symlink just to satisfy Arch's usual packaging layout guidance.

## Proposed changes

1. **Restore the vendor filesystem layout in the Arch package.**
   - Change `package()` in `PKGBUILD` to place the Brother `Printer` tree under `${pkgdir}/usr/local/Brother/Printer`.
   - Revert the relocation-specific `/usr/share/brother/printer` paths in the patch so the generated wrapper and model filter use the vendor's `/usr/local/Brother/Printer` paths.
   - Keep the patch's removal of the vendor installer's system-changing behavior (such as configuring a queue, restarting services, and changing permissions broadly). Do not re-enable the vendor installer as a package-install mechanism.

2. **Grant only the CUPS group the required RC access.**
   - The source RPM includes `inf/brMFC7860DWrc`; it is not generated on first print. The RPM records that file as mode `0666` and the `inf` directory as `0755`.
   - In the packaged tree, set the RC file to a restrictive group-writable mode and assign it to the CUPS group. Make the `inf` directory group-writable as well, since the helper renames the RC file within that directory.
   - Keep ownership root-controlled and avoid world-writable permissions. Verify the CUPS group name and the final package metadata in the target Arch environment.

3. **Document the concurrency concern for later.**
   - Add a README documenting that per-job configuration mutates one shared RC file. Defer locking for now, but record that concurrent jobs could race and that a future change should investigate serializing the helper and filter use of the shared file.

4. **Update package metadata.**
   - Increment `pkgrel` and regenerate/update `.SRCINFO` to match the PKGBUILD.
   - Update any source-patch checksum affected by edits.

5. **Handle migration from the manual compatibility symlink.**
   - The current local workaround may be a symlink at `/usr/local/Brother/Printer/MFC7860DW` pointing into `/usr/share`.
   - Before installing the revised package, remove or replace that symlink so the package can install the real vendor tree at the destination. Do not silently overwrite user data; document the migration requirement.

## Verification

- Build and run the package checks (`makepkg --verifysource --syncdeps --cleanbuild` as appropriate) and inspect the resulting package file list, ownership, and modes.
- Use a disposable Arch chroot to install the built package and test as the `cups` user. Confirm `brcupsconfig4` can update the RC file and that the model filter can read it.
- Exercise simplex and duplex option updates in sequence, checking the RC contents after each invocation. No printer queue or physical print is needed; redirect filter output to a file or `/dev/null`.
- Confirm that no package files are installed under `/usr/share/brother/printer`, no `/usr/local` compatibility symlink is required, and no vendor post-install action configures a queue or restarts CUPS.
