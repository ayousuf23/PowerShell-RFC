---
RFC:
Author: Abdullah Yousuf
Status: Draft
SupercededBy: 
Version: 1.0
Area: Archive
Comments Due: 6/30/2022
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

    As a PowerShell user,
    I can create an archive that preserves the relative path structure, so that I can keep track of which folders the contents of the archive came from and so that I can replicate the same structure on other computers.

Relative path structure refers to the hierarchial structure of a path relative to the current working directory.

    As a PowerShell user,
    I can filter what files to include in an archive,
    so that I store the necessary files only.

    As a PowerShell user,
    I can filter what files to extract from an archive,
    so that I obtain the necessary files only.
## User Experience

### Compress-Archive

Parameter sets:

```powershell
Compress-Archive [-Path] <string[]> [-DestinationPath] <string> [-CompressionLevel {Optimal | NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

Compress-Archive [-Path] <string[]> [-DestinationPath] <string> -Force [-CompressionLevel {Optimal | NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

Compress-Archive [-Path] <string[]> [-DestinationPath] <string> -Update [-CompressionLevel {Optimal |NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> [-CompressionLevel {Optimal | NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> -Force [-CompressionLevel {Optimal | NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> -Update [-CompressionLevel {Optimal | NoCompression | Fastest | SmallestSize}] [-PassThru] [-Format {zip | tar | targz}] [-Filter <string>] [-Flatten] [-WhatIf] [-Confirm] [<CommonParameters>]

```


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
Compress-Archive MyFolder destination.zip -Filter *.txt
```

A zip archive is created with the name `destination.zip`. Notice the cmdlet sees the `.zip` extension and uses the `zip` format. The only files in the archive are those with the `.txt` extension.

```powershell
Compress-Archive MyGrandparentFolder/MyParentFolder/MyFolder destination.zip 
```

A zip archive is created with the name `destination.zip`. The archive is structured as:

```
destination.zip
|---MyGrantparentFolder
    |---MyParentFolder
        |---MyFolder
            |---*
```
The archive preserves the relative structure of the input path.

```powershell
Compress-Archive MyGrandparentFolder/MyParentFolder/MyFolder destination.zip -Flatten
```

A zip archive is created with the name `destination.zip`. The archive is structured as:

```
destination.zip
|---MyFolder
    |---*
```
The archive **does not** preserve the relative structure of the input path since `-Flatten` is specified.

### Expand-Archive

Parameter sets:

```powershell
Expand-Archive [-Path] <string> [[-DestinationPath] <string>] [-Force] [-PassThru] [-Filter <string>] [-WhatIf] [-Confirm] [<CommonParameters>]

Expand-Archive [[-DestinationPath] <string>] -LiteralPath <string> [-Force] [-PassThru] [-Filter <string>] [-WhatIf] [-Confirm] [<CommonParameters>]
```

```powershell
Expand-Archive MyArchive.tar.gz
```

A new folder is created in the current working directory called `MyArchive`. The files and directories contained in the archive is added to that folder.

```powershell
Expand-Archive MyArchive.tar.gz -Format zip
```
This does the same as above except that the archive is forcably interpreted as a zip archive rather than a compressed tar archive.

```powershell
Expand-Archive MyArchive.tar.gz -Filter *.txt
```
This does the same as above except that only the files ending with `.txt` extension are extracted.

## Specification

### Compress-Archive

#### `-Format` parameter

`Compress-Archive` has an optional parameter called `-Format`, which accepts one of three options: `zip`, `tar`, or `targz`. This parameter supports tab completion.

The format of the archive is determined by the extension of `-DestinationPath` required parameter or the value supplied to the `-Format` parameter. If the extension is `.zip`, `.tar`, or `.tar.gz`, the appropriate archive format is determined by the command. 

Example:
```powershell
Compress-Archive MyFolder destination.tar
```
The tar format is chosen for the archive since destination.tar has the `.tar` extension.

In the case when both `-DestinationPath` parameter has a matching extension and `-Format` parameter are specified, the `-Format` parameter takes precedence. 

Example:
```powershell
Compress-Archive MyFolder destination.tar -Format zip
```
The zip format is chosen for the archive since the `-Format` parameter takes precedence over the extension of destination.tar. Note that `.zip` is appended to the name of the archive, which becomes `destination.tar.zip`.  

When the `-DestinationPath` parameter does not have a matching extension and the `-Format` parameter is not specified, by default the zip format is chosen.  

#### **Relative Path Structure Preservation**

When valid path(s) is supplied to the `-Path` parameter, the relative structure of the path is preserved as long as the path is not absolute, and does not contain ~ or .. (the path can contain other wildcard characters such as *, [, ], ?, and `).

Example: `Compress-Archive C:\Users\user\Documents destination.zip`
creates an archive with the structure:

```
destination.zip
|---Documents
```
The grandparent and parent directories User and user are not preserved in the archive because the path is absolute.

Similarly, `Compress-Archive ~\Documents destination.zip` and `Compress-Archive C:\Users\user\..\Documents destination.zip` exhibit the same behavior.

The relative path(s) supplied to the `-Path` parameter must be relative to the current working directory.

The `-Flatten` switch parameter can be used to prevent relative path structure preservation. The `-Flatten` parameter keeps the archive structure flat.

Example: `Compress-Archive Documents\MyFile.txt destination.zip`
creates an archive with the structure:

```
destination.zip
|---Documents
    |---MyFile.txt
```

When the `-Flatten` parameter is specified to the cmdlet above, the archive has the following structure:

```
destination.zip
|---MyFile.txt
```

#### `-Filter` parameter

When the `-Filter` parameter is supplied with a value, the cmdlet stores files in the archive that match the filter only. Directories are still archived unless they are empty or do not contain files that match the filter.

If the path is a directory, the filter applies to all files in the directory no matter how deep they are in the hierarchy.

The filter accepts standard PowerShell wildcard characters. The filter performs matching based on the file name, so filtering based on a path does not work.

Example: `Compress-Archive Folder destination.zip -Filter /ChildFolder/*` will output an empty archive. 

Example: `Compress-Archive Folder destination.zip -Filter s*` outputs an archive whose files start with s only.

The filter paramater preserves the archive structure in the output -- directories in the input path(s) will be added to the archive as long as they contain at least one file after applying the filter.

Example: Suppose we want to archive `Folder` which has the following structure:

```
Folder
|---ChildFolder1
    |---file.txt
|---ChildFolder2
    |---file.md
```

`Compress-Archive Folder destination.zip -Filter *.txt` creates an archive with the followng structure:

```
destination.zip
|---Folder
    |---ChildFolder1
        |---file.txt
```

### Expand-Archive

#### `-Format` parameter

The `-Format` parameter accepts one of three options: `zip`, `tar`, or `targz`. This parameter supports tab completion.

The format of the archive is determined by the extension of `-Path` or `-LiteralPath` (one of which is required) parameter or the value supplied to the `-Format` parameter. If the extension is `.zip`, `.tar`, or `.tar.gz`, the appropriate archive format is determined by the command. 

Example:
```powershell
Expand-Archive -Path archive.tar
```
The tar format is chosen for the archive since archive.tar has the `.tar` extension.

In the case when either `-Path` or `-LiteralPath` parameter has a matching extension and `-Format` parameter is specified, the `-Format` parameter takes precedence. 

Example:
```powershell
Expand-Archive -Path archive.tar -Format zip
```
The archive is interpreted as a zip archive since the `-Format` parameter takes precedence over the extension of archive.tar. 


When the `-Path` or `-Literalpath` parameters do not have a matching extension and the `-Format` parameter is not specified, by default the archive is interpeted as a zip archive. 

For `Expand-Archive`, when an archive format is determined by the cmdlet that does not match the format of the archive passed to it, a terminating error will be thrown (e.g. if `-Format zip` is specified for a tar archive).

#### `-Filter` parameter

When the `-Filter` parameter is supplied with a value, the cmdlet extracts files in the archive that match the filter only.

The filter applies to all files in the archive no matter how deep they are in the hierarchy.

The filter accepts standard PowerShell wildcard characters. The filter performs matching based on the file name, so filtering based on a path does not work.

Example: `Expand-Archive archive.zip -Filter /ChildFolder/*` will create a folder with the name `archive` whose contents will be empty because no filename in the archive matches the filter.

Example: `Expand-Archive archive.zip -Filter s*` will create a folder with the name `archive` whose files start with s.

The filter paramater preserves the archive structure in the output -- directories in the archive will be created in the output as long as they contain at least one file after applying the filter.

Example: Suppose we want to expand `archive.zip` which has the following structure:

```
archive.zip
    |---ChildFolder1
        |---file.txt
    |---ChildFolder2
        |---file.md
```

`Expand-Archive archive.zip -Filter *.txt` creates a folder with the followng structure:

```
archive
|---ChildFolder1
    |---file.txt
```


#### Default output location for `Expand-Archive`

Currently, when `Expand-Archive archive.zip` is called, the contents of the archive are added to the current working directory. This is unintuitive because the user does not necessarily know what the contents of the archive are. An archive may contain a top-level folder with a name that is different than the filename of the archive.

The solution is to create a directory in the current working directory with the name of the archive excluding the file extension (e.g. `archive`) and the contents of the archive are added to that directory.

If the user wants to put the contents of the archive in the current working directory (i.e., same behavior as before), supplying the path of the current working directory to the `-DestinationPath` parameter exhibits such behavior.


## Downlevel Support

The enhancements discussed in this RFC depend on System.Formats.Tar v7 and System.IO.Compression .NET Framework 4.8/.NET Core 1.0+.

The current plan is to support PowerShell 7. Supporting Windows PowerShell requires additional work related to  loading the correct assembly because the RFC requires a newer version of System.IO.Compression, and Windows PowerShell depends on an older version of it.

## Alternate Proposals and Considerations

None