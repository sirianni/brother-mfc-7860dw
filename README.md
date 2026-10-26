# brother-mfc-7860dw

This package adapts Brother's legacy RPM driver for the MFC-7860DW. The vendor driver expects its files under `/usr/local/Brother/Printer`, so this package preserves that layout instead of relocating the tree. The vendor installer is not run during packaging; the PPD and CUPS filter are installed as package files. The packaged default RC configuration enables duplex printing and selects Letter paper.

## Runtime configuration permissions

`brcupsconfig4` applies job options by renaming and rewriting `inf/brMFC7860DWrc`. The package therefore makes that RC file writable by the `cups` group and makes its containing `inf` directory group-writable. The file remains root-owned and is not world-writable.

The RC file is shared mutable state. Locking is not implemented in this package version. Concurrent print jobs may race while changing or reading settings; serializing the helper and filter use of this file is a future consideration.

## Migration from the old relocated package

If upgrading from a version that used `/usr/share/brother/printer`, check for the manually created compatibility symlink at `/usr/local/Brother/Printer/MFC7860DW`. Remove that symlink before installing this package's real vendor tree at the same path. Verify it is the expected symlink before removing it; do not remove a real directory or unrelated local data.

## Non-printing validation

The helper and wrapper can be exercised as the `cups` user with test PostScript input, sending the wrapper's output to a regular file rather than a CUPS backend. This checks option application and filter output without queuing a job to a physical printer.
