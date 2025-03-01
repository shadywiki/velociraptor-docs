---
title: Windows.Events.ProcessCreation
hidden: true
tags: [Client Event Artifact]
---

Collect all process creation events.

This artifact relies on WMI to receive process start events. This
method is not as good as kernel mechanism used by Sysmon. It is more
reliable to use Sysmon instead via the
Windows.Sysinternals.SysmonLogForward artifact instead.


```yaml
name: Windows.Events.ProcessCreation
description: |
  Collect all process creation events.

  This artifact relies on WMI to receive process start events. This
  method is not as good as kernel mechanism used by Sysmon. It is more
  reliable to use Sysmon instead via the
  Windows.Sysinternals.SysmonLogForward artifact instead.

type: CLIENT_EVENT

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'
    query: |
      -- Add a small delay to allow the process tracker to catch up
      -- for enrichments.
      LET Delayed = SELECT * FROM delay(query={
         SELECT * FROM wmi_events(
             query="SELECT * FROM Win32_ProcessStartTrace",
             wait=5000000,   // Do not time out.
             namespace="ROOT/CIMV2")
      }, delay=2)

      // Convert the timestamp from WinFileTime to Epoch.
      SELECT timestamp(winfiletime=atoi(string=Parse.TIME_CREATED)) as Timestamp,
          Parse.ParentProcessID as PPID,
          Parse.ProcessID as PID,
          Parse.ProcessName as Name,
          process_tracker_get(id=Parse.ProcessID).Data.CommandLine AS CommandLine,
          process_tracker_get(id=Parse.ParentProcessID).Data.CommandLine AS ParentCommandLine,
          join(array=process_tracker_callchain(id=Parse.ProcessID).Data.Name,
               sep=" <- ") AS CallChain
      FROM Delayed

```
