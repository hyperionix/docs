---
layout: default
title: Advanced Testing
nav_order: 5
parent: Packages Step By Step
permalink: package-5
---

# Advanced testing
After making sure that the probe has worked correctly we can try to test it with real process. Lets inject the probe into explorer.exe process.

NOTE: your antivirus could block this operation.

```bat
.\bin\hdk --inject "My Process Created" --process "explorer.exe"
```

Try to start some processes. You should see events in the `hdk` console. Try to start notepad process. This operation should be blocked according to our probe.
