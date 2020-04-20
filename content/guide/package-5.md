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
.\bin\hdk --inject "File Delete" --process "explorer.exe"
```

Try to delete some files (not moving to the Recycle Bin, namely removing. Use Shift + Delete for this). You should see events in the `hdk` console. 

Coming soon
{: .label .label-yellow }