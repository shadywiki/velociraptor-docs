---
title: Windows.System.Signers
hidden: true
tags: [Client Artifact]
---

This artifact searches for all signed files and stacks them by signer.


```yaml
name: Windows.System.Signers
description: |
   This artifact searches for all signed files and stacks them by signer.

parameters:
   - name: ExecutableGlobs
     default: C:/Windows/**/*.{dll,exe}
   - name: ShowAllSigners
     description: When checked we show all signed files instead of stacking them.
     type: bool

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'

    query: |
        LET results = SELECT FullPath, count() AS Count,
               parse_pe(file=FullPath).Authenticode.Signer.Subject AS Signer
        FROM glob(globs=ExecutableGlobs)
        WHERE Signer

        SELECT * FROM if(condition=ShowAllSigners,
        then={
            SELECT FullPath, Signer FROM results
        }, else={
            SELECT Count, Signer FROM results
            GROUP BY Signer
            ORDER BY Count DESC
        })

```
