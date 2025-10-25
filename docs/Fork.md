# AvailabilityVersions fork

## Overview

This is a community-maintained fork of Apple's AvailabilityVersions repo, which provides version number definitions and availability macros for Apple platforms.

For more information the branching scheme, please see [docs/PatchBranchingScheme.md](docs/PatchBranchingScheme.md).

### Background

Apple's official AvailabilityVersions distribution (last updated as version 155) has not had its `availability.dsl` file updated since iOS 17.3. This causes the open-source dyld to fail identifying newer OS versions, as the `<dyld/VersionMap.h>` and `sVersionMap` data structures remain outdated.

This fork reconstructs the missing availability data from dyld's internal version mappings to maintain compatibility with current and future OS releases.

## Reconstruction Process

Since Apple no longer updates the DSL file, we reconstruct `availability.dsl` entries by extracting version data from dyld binaries found in OS releases.

### Step 1: Extract Version Set Data from dyld Binary

Use a disassembler (e.g., Hopper, IDA Pro, Binary Ninja) or `otool` to locate the `sVersionMap` symbol in the dyld binary (/usr/lib/dyld):

The `VersionSetEntry` structures appear as arrays of hexadecimal values:

```
__ZN5dyld3L11sVersionMapE:  // dyld3::sVersionMap
struct VersionSetEntry {
    0x7e80600,    // .set
    0xf0600,      // .macos
    0x120600,     // .ios
    0xb0600,      // .watchos
    0x120600,     // .tvos
    0x90600,      // .bridgeos
    0x180600,     // .driverkit
    0x20600       // .visionos
}
```

**Field Order** (as defined in `VersionMap.h`):
1. `set` - Version set identifier
2. `macos` - macOS version
3. `ios` - iOS version  
4. `watchos` - watchOS version
5. `tvos` - tvOS version
6. `bridgeos` - bridgeOS version
7. `driverkit` - DriverKit version
8. `visionos` - visionOS version

### Step 2: Decode Version Numbers

Version numbers use the format `0x00MMMMRRRR` where:
- `MMMM`: Major.Minor version in hex
- `RRRR`: Revision/patch version in hex

Examples:
- `0x007e80600` decodes to `2024.6.0` (0x7e8 = 2024, 0x06 = 6, 0x00 = 0)
- `0x00120600` decodes to `18.6.0` (0x12 = 18, 0x06 = 6, 0x00 = 0)
- `0x000f0600` decodes to `15.6.0` (0x0f = 15, 0x06 = 6, 0x00 = 0)

**Important**: Set versions for 2024 releases use the simplified format `2024.N.0`:
- `fall_2024`: `2024.0.0` → `0x007e80000`
- `2024_SU_B`: `2024.1.0` → `0x007e80100`
- `2024_SU_C`: `2024.2.0` → `0x007e80200`
- `2024_SU_G`: `2024.6.0` → `0x007e80600`

Platform versions decode normally using their actual OS version numbers.

### Step 3: Determine Darwin Kernel Version

The third parameter (uversion) in each set definition corresponds to the Darwin kernel version. Query from:
- Online resources: [The Apple Wiki - Kernel Versions](https://theapplewiki.com/wiki/Kernel#Versions)
- System queries: `uname -r` on target OS
- XNU source releases

Example mappings:
- macOS 15.0 / iOS 18.0 → Darwin 24.0.0
- macOS 15.1 / iOS 18.1 → Darwin 24.1.0
- macOS 15.6 / iOS 18.6 → Darwin 24.6.0

### Step 4: Add DSL Entries

Follow the established naming convention for set names:

```plaintext
set         2024_SU_G           2024.6.0     24.6.0
version     macos               15.6
version     ios                 18.6
version     tvos                18.6
version     watchos             11.6
version     bridgeos            9.6
version     driverkit           24.6
version     visionos            2.6
```

**Note**: Set names follow previous conventions (e.g., `fall_YYYY`, `YYYY_SU_X`) and may differ from Apple's internal naming.

### Example: Complete Extraction Workflow

Given this binary data from dyld:
```
0x7e80600, 0xf0600, 0x120600, 0xb0600, 0x120600, 0x90600, 0x180600, 0x20600
```

1. **Decode versions**:
   - Set: `2024.6.0`
   - macOS: `15.6.0`
   - iOS: `18.6.0`
   - watchOS: `11.6.0`
   - tvOS: `18.6.0`
   - bridgeOS: `9.6.0`
   - DriverKit: `24.6.0`
   - visionOS: `2.6.0`

2. **Lookup Darwin version**: `24.6.0` (from Apple Wiki or system)

3. **Create DSL entry**:
```plaintext
set         2024_SU_G           2024.6.0     24.6.0
version     macos               15.6
version     ios                 18.6
version     tvos                18.6
version     watchos             11.6
version     bridgeos            9.6
version     driverkit           24.6
version     visionos            2.6
```

## File Structure

- `availability.dsl` - Domain-specific language defining platform versions and sets
- `availability` - Python script for querying and preprocessing version data
- `templates/` - Header templates with preprocessing macros
- Generated outputs:
  - `<dyld/VersionMap.h>` - Version set mapping table
  - `<Availability.h>` - Availability macros and version defines

## Building

The build system uses CMake, with a Makefile wrapper for compatibility:

```bash
make install
```

Build process:
1. Self-preprocess `availability` script to embed DSL content
2. Preprocess all template files using the embedded DSL data
3. Install processed files to `DSTROOT`

## Usage

Query platform versions:

```bash
# List all iOS versions
/usr/local/libexec/availability --ios

# List all version sets (YAML format)
/usr/local/libexec/availability --sets

# Preprocess a template file
/usr/local/libexec/availability --preprocess input.h output.h
```

## Contributing

When adding new OS versions:

1. Extract version data from the latest dyld binary using a disassembler
2. Decode hexadecimal versions to decimal format
3. Verify Darwin kernel version from official sources
4. Follow existing naming conventions for set names
5. Maintain chronological order in `availability.dsl`
6. Test generated headers for correctness

## License

This project maintains compatibility with Apple's original licensing terms. See individual file headers for details.

## See Also

- [dyld Project](https://github.com/apple-oss-distributions/dyld)
- [XNU Kernel](https://github.com/apple-oss-distributions/xnu)
- [The Apple Wiki - Kernel Versions](https://theapplewiki.com/wiki/Kernel#Versions)
