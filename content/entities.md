---
layout: default
title: Entities
permalink: entities
nav_order: 4
---

# Entities

Entities are special objects which designed with capability to be monitored and analyzed with Hyperionix Entity State Machine and/or track them in event storages. Each entity has Entity ID `eid` field in format 
```
eid://<EntityType>:<UID>
```
Entity includes information about the object it describes. This information could be extended inside your probe. Also entities could include other entities, you may notice that process entity includes file entity for main executable. Entity for the process could look like this:
```json
{
    "backingFile": {
        "attributes": 32,
        "certificate": {
            "signers": [
                "C=US, S=ca, L=Mountain View, O=Google LLC, CN=Google LLC"
            ],
            "status": "Trusted"
        },
        "eid": "eid://file:3751977393",
        "path": {
            "basename": "chrome",
            "dir": "\\Program Files (x86)\\Google\\Chrome\\Application\\",
            "drive": "C:",
            "ext": ".exe",
            "full": "C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe"
        },
        "resources": {
            "company": "Google LLC",
            "description": "Google Chrome",
            "originalFileName": "chrome.exe",
            "version": "80.0.3987.163"
        },
        "size": 1712112,
        "times": {
            "change": 1587144455,
            "create": 1547537069,
            "read": 1587146015,
            "write": 1585786077
        }
    },
    "bitness": 64,
    "commandLine": "\"C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe\" --type=utility --field-trial-handle=2072,14393559574579131839,7586634091475304641,131072 --lang=en-US --service-sandbox-type=utility --enable-audio-service-sandbox --mojo-platform-channel-handle=5128 --ignored=\" --type=renderer \" /prefetch:8",
    "createTime": 1587146164,
    "domain": "DESKTOP-4FH1109",
    "eid": "eid://process:84961587146164",
    "integrityLevel": "LOW",
    "pid": 8496,
    "policies": {
        "alr": 1,
        "binarySignature": 0,
        "dep": 1,
        "prohibitDynamicCode": 0
    },
    "ppid": 8588,
    "user": "user"
}
```


