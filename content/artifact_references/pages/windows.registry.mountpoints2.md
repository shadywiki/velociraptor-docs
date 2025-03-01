---
title: Windows.Registry.MountPoints2
hidden: true
tags: [Client Artifact]
---

This detection will collect any items in the MountPoints2 registry key.
With a "$" in the share path. This key will store all remotely mapped
drives unless removed so is a great hunt for simple admin $ mapping based
lateral movement.


```yaml
name: Windows.Registry.MountPoints2
description: |
    This detection will collect any items in the MountPoints2 registry key.
    With a "$" in the share path. This key will store all remotely mapped
    drives unless removed so is a great hunt for simple admin $ mapping based
    lateral movement.

author: Matt Green - @mgreen27

precondition: SELECT OS From info() where OS = 'windows'

parameters:
 - name: KeyGlob
   default: Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\*
 - name: MountPointFilterRegex
   type: regex
   default: "\\$"

sources:
 - query: |
        SELECT regex_replace(
            source=OSPath.Basename,
            re="#",
            replace="\\") as MountPoint,
          Mtime as ModifiedTime,
          Username,
          OSPath.DelegatePath as Hive,
          OSPath.Path as Key
        FROM Artifact.Windows.Registry.NTUser(KeyGlob=KeyGlob)
        WHERE FullPath =~ MountPointFilterRegex

```
