---
RFC:
Author: Abdullah Yousuf
Status: Draft
SupercededBy: 
Version: 1.0
Area: Archive
Comments Due: <Date for submitting comments to current draft (minimum 1 month)>
Plan to implement: Yes
---

# Archive Module Version 2

This RFC proposes new features and changes to existing behavior for the existing Microsoft.PowerShell.Archive module.

## Motivation

    As a PowerShell user,
    I can create archives larger than 4GB,
    so that I store large files in a portabable and easily compressable format.

Currently, archives are limited to a size of approximately 4GB. 

    As a PowerShell user,
    I can create tar archives,
    so that I can reliably deliver files to UNIX machines, which have tar support pre-installed.



    As a PowerShell user,
    I can create compressed tar archives,
    so that I can reliably store and deliver content while taking up less storage space.


## User Experience

Example of user experience with example code/script.
Include example of input and output.

```powershell
Compress-Archive MyFolder destination -Format zip
```

A zip archive is created with the name `destination.zip`. Note that the `.zip` file extension is appended automatically to the destination path.

```powershell
Compress-Archive MyFolder destination -Format tar
```

A tar archive is created with the name `destination.tar`. Note that the `.tar` file extension is appended automatically to the destination path.

```powershell
Compress-Archive MyFolder destination.tar.gz -Format tar.gz
```

A tar.gz archive is created with the name `destination.tar.gz`.

```powershell
Expand-Archive MyArchive.tar.gz
```

A new folder is created in the current working directory called `MyArchive`. The files and directories contained in the archive is added to that folder. See specification for collision details.

```powershell
Expand-Archive MyArchive.tar.gz -Format zip
```
This does the same as above except that the archive is forcably interpreted as a zip archive.

## Specification

### `-Format` parameter

`Compress-Archive` and `Expand-Archive` will have an optional parameter called `-Format`, which accepts one of three options: `zip`, `tar`, or `tar.gz`.

For `Compress-Archive`, the format of the archive will be determined by the extension of `-DestinationPath` required parameter. If the extension is `.zip`, `.tar`, or `.tar.gz`, the appropriate archive format is determined by the command. If the extension does not match any of those previously mentioned, by default the `zip` format is chosen.

For `Expand-Archive`, the extension of the `-Path` or `-LiteralPath` (either one is required) parameter will be looked at and the same behavior as above is exhibited.

In the case when both a valid extension and `-Format` parameter are specified, the `-Format` parameter takes precedence. 

For `Expand-Archive`, when an archive format is determined by the command that does not match the format of the archive passed to it, a terminating error will be thrown.

### Default output location for `Expand-Archive`

Currently, when `Expand-Archive archive.zip` is called, the contents of the archive are added to the current working directory. This is unintuitive because the user does not necessarily know what the contents of the archive are. The solution is to create a directory in the current working directory with the name `archive` (name of the archive excluding file extension) and the contents of the archive will be added to that directory.

If the user wants to put the contents of the archive in the current working directory (i.e., same behavior as before), supplying the path of the current working directory to the `-DestinationPath` parameter will exhibit such behavior.

- Make update throw an error if archive does not exist



## Alternate Proposals and Considerations

None

# Personal notes

- Discuss downlevel support
- Include all parameter sets
- Remove zip64
- Improve user stories
- Split into 2 sections
- enum format
- Flatten parameter
- Keep relative directory path
- Filter