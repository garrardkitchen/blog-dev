---
title: "How to Write Clipboard to File"
date: 2022-09-26T17:21:46+01:00
tags: [clipboard, powershell, Microsoft.PowerShell.Management]
---

Have you ever wondered how to persist your clipboard to disk?  Wonder no more...

To get started, copy a random sentence into your clipboard like so:

```powershell
Set-Clipboard -Value "This is being copied into clipboard"
```

Let's confirm the contents of the clipboard, like so:

```powershell
Get-Clipboard
```

_output_:
```
This is being copied into clipboard
```

We'll now persist the same cliboard to a file, like so:

```powershell
Get-Clipboard | Out-File -FilePath clipboard.txt
```

To confirm the contents of this file, we type:

```powershell
Get-Content .\clipboard.txt
```

_output_:
```
This is being copied into clipboard
```

Let's append something else into the clipboard like so:

```powershell
Set-Clipboard  "appended!" -Append
```

And finally, we confirm that this text has indeed been copied into the clipboard like so:

```powershell
Get-Clipboard
```

_output_:

```
This is being copied into clipboard
appended!
```

There you have it, a short post on how to obtain and set the clipboard using Powershell commands.

**Module**: Microsoft.PowerShell.Management